# Notebook 控制器

本控制器允许用户创建“Notebook”类型的自定义资源（jupyter notebook）.
我们原来是用 jsonnet 和 metacontroller 来写的，后面迁移到 golang 和
Kubebuilder。参考[这里的讨论](https://github.com/kubeflow/kubeflow/issues/2269)。

## 规定

用户需要为 jupyter notebook 定义 PodSpec。
示例：

```
apiVersion: kubeflow.org/v1alpha1
kind: Notebook
metadata:
  name: my-notebook
  namespace: test
spec:
  template:
    spec:  # Your PodSpec here
      containers:
      - image: gcr.io/kubeflow-images-public/tensorflow-1.10.1-notebook-cpu:v0.3.0
        args: ["start.sh", "lab", "--LabApp.token=''", "--LabApp.allow_remote_access='True'",
               "--LabApp.allow_root='True'", "--LabApp.ip='*'",
               "--LabApp.base_url=/test/my-notebook/",
               "--port=8888", "--no-browser"]
        name: notebook
      ...
```

必须的字段包括 `containers[0].image` 和 (`containers[0].command` 或 `containers[0].args`)。
这里是，用户需要定义的运行指令。

如果未指定，所有其他字段将填充默认值。

## 环境变量
|变量 | 描述 |
| --- | --- |
|ADD_FSGROUP| 如果值未 true 或未设置，fsGroup: 100 将用于 pod 的安全上下文。如果该值存在并设置为 false，它将禁止自动将 fsGroup: 100 添加到 pod 的安全上下文中。|
|DEV| 如果该值为 false 或未设置，则将使用 Notebook 控制器的默认实现。 如果管理员想要在本地计算机上使用自定义实现，则应将此值设置为 true。 |


   
## 命令行参数

`metrics-addr`: 绑定的指标端口。 默认值是 `:8080`.

`enable-leader-election`: 为控制器管理器启用领导者选举。 启用此功能将确保只有一个活动控制器管理器。默认值为 `false`。

## 实现细节

这部分是 WIP，因为我们仍在开发中。

在底层，控制器创建一个 StatefulSet 来运行 Notebook 实例，并为其创建一个 Service。

## 贡献

[https://www.kubeflow.org/docs/about/contributing/](https://www.kubeflow.org/docs/about/contributing/)

### 开发环境

要在 `notebook-controller` 之上开发，你的环境必须遵从：

- [go](https://golang.org/dl/) 版本 v1.15+.
- [docker](https://docs.docker.com/install/) 版本 17.03+.
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 版本 v1.11.3+.
- [kustomize](https://sigs.k8s.io/kustomize/docs/INSTALL.md) v3.1.0+
- 接入 Kubernetes v1.11.3+ 集群.
- [kubebuilder](https://book.kubebuilder.io/quick-start.html#installation)

为了使自定义 Notebook 控制器能够在本地计算机上运行，
管理员需：

1. 设置副本数为零： 
   ``` 
   kubectl edit deployment notebook-controller-deployment -n=kubeflow
   ```
2. 通过在本地计算机上执行来允许控制器通过将流量代理到 Notebook Services：
   ```
   kubectl proxy
   ```
## 代办

- e2e 测试 (我们已有一个测试 jsonnet-metacontroller 的控制器，我们应该让它在这个控制器上运行)
- `status` 字段应在出现任何错误时反映出来。参见 [#2269](https://github.com/kubeflow/kubeflow/issues/2269).
- Istio 实现 (控制器会生成 istio 资源来保障每个用户 notebook 安全)
- CRD [验证](https://github.com/kubeflow/kubeflow/blob/master/kubeflow/jupyter/notebooks.schema)
- `ttlSecondsAfterFinished`: 这是在原本的 jsonnet 控制器章节，但已经不再使用。 我想我们会在闲置时间清理掉它？
- 增加更多关于构建，部署以及本地测试的贡献说明。
- 一个安装所有依赖的脚本。
