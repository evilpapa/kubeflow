## 目标
我们需要一种方法来注入通用数据 (env vars, volumes) 到 pod (比如 notebooks)中。
参见 [讨论](https://github.com/kubeflow/kubeflow/issues/2641).
K8s 有 [PodPreset](https://v1-19.docs.kubernetes.io/docs/concepts/workloads/pods/podpreset/) 类似用例的资源，然而它还处在 alpha 阶段。
K8s [admission-controller](https://godoc.org/k8s.io/api/admissionregistration/v1beta1#MutatingWebhookConfiguration) 和 CRD 可用于实现 PodPreset，如[此处所示](https://github.com/jpeeler/podpreset-crd)。
我们借用了这个 PodPreset 实现，为 Kubeflow 进行了定制，并将其重命名为 PodDefault 以避免混淆。
该代码未直接使用，因为 Kubeflow 的 PodDefault 控制器用例略有不同。
事实上，Kubeflow 中的 PodDefault 被定义为没有自定义控制器的 CRD (相对于[此](https://github.com/jpeeler/podpreset-crd))。

## 如何工作
以下是在 Kubeflow 中如何使用它的工作流程：

1. 用户创建 PodDefault 货单，它描述了额外的运行时要求 (如 volume, volumeMounts, 环境变量) 要在 Pod 创建时进行注入。
PodDefaults 使用标签选择器（label selectors）来指定那些应用了 PodDefault 的 Pod。
PodDefaults 是命名空间范围的，例如，它们能在命名空间内被应用/查看。
作为示例，以下货单声明了在 `kubeflow` 空间内声明了一个 `PodDefault` 来添加密钥 ```gcp-secret``` 到指定空间的 Pod 中。

```
apiVersion: "kubeflow.org/v1alpha1"
kind: PodDefault
metadata:
  name: add-gcp-secret
  namespace: kubeflow
spec:
 selector:
  matchLabels:
    add-gcp-secret: "true"
 desc: "add gcp credential"
 volumeMounts:
 - name: secret-volume
   mountPath: /secret/gcp
 volumes:
 - name: secret-volume
   secret:
    secretName: gcp-secret
```
1.  Kubeflow 组件，负责(如, notebook 控制器)创建 pod 在需要时向 Pod 添加一些可用的 PodDefault 标签。
针对 Jupyter notebooks，例如，Notebook UI 询问用户哪些 PodDefault 需要应用到 notebook pods (参看[讨论](https://github.com/kubeflow/kubeflow/issues/2992))。
Notebook-controller 之后会将相应的 PodDefault 标签添加到 Notebook Pod。


1. 通常 [Admission webhook 控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)，
拦截去往 Kubernetes API server 的请求，并可以修改，或者验证请求。
这里实现了admission webhook来根据可用的 PodDefaults 修改pod。
当收到 Pod 创建请求时，准入 Webhook 会查找与 Pod 标签匹配的可用 PodDefaults。
然后，它根据 PodDefault 的规范改变 Pod 规范。
对于上面的 PodDefault，当带有标签 `add-gcp-secret:"true"` 的 pod 创建请求到来时，它会附加卷和volumeMounts
按照 PodDefault 规范中的描述添加到 pod。

## Webhook 配置
定义 [MutatingWebhookConfiguration](https://godoc.org/k8s.io/api/admissionregistration/v1beta1#MutatingWebhookConfiguration)，
示例：

```
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingWebhookConfiguration
metadata:
  name: gcp-cred-webhook
webhooks:
  - name: gcp-cred-webhook.kubeflow.org
    clientConfig:
      service:
        name: gcp-cred-webhook
        namespace: default
        path: "/add-cred"
      caBundle: "..."
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
```

这规定
1. 当 pod 被创建 (查看 `rules`),
1. 请求 webhook 服务 `gcp-cred-webhook.default` 的 `/add-cred`  路径(查看 `clientConfig`)


### Webhook 实现
webhook 是一个处理来自(`/add-cred` 如上)配置路径请求的服务。
请求和响应类型均未 [AdmissionReview](https://godoc.org/k8s.io/api/admission/v1beta1#AdmissionReview)

## 参考
1. [K8S PodPreset](https://v1-19.docs.kubernetes.io/docs/concepts/workloads/pods/podpreset/)
1. https://github.com/jpeeler/podpreset-crd
1. https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/
1. https://github.com/kubernetes/kubernetes/tree/v1.13.0/test/images/webhook
1. https://github.com/morvencao/kube-mutating-webhook-tutorial
1. 如何自我签名: [链接](https://github.com/kubernetes/kubectl/issues/86)
1. caBundle 中要放什么: [讨论](https://github.com/kubernetes/kubernetes/issues/61171)
