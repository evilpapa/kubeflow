# Kubeflow 配置文件

Kubeflow Profile CRD 旨在解决多用户 kubernetes 集群内的访问管理。

配置文件访问管理提供命名空间级别的隔离，基于：

- Kubernetes RBAC
- Istio 授权策略

**配置文件 CRD 管理的资源：**

每个 profile CRD 将管理一个命名空间（与 profile CRD 同名）并拥有一个所有者。
具体来说，每个 profile CRD 将管理以下资源：

- 为 profile 所有者保留的命名空间。
- K8s RBAC RoleBinding `namespaceAdmin`: 使 profile 所有者成为命名空间管理员，允许通过 k8s API (kubectl) 访问上述命名空间。
- Istio 命名空间范围的 ServiceRole `ns-access-istio`: 允许通过 Istio 路由访问目标命名空间中的所有服务。
- Istio 命名空间范围的 ServiceRoleBinding `owner-binding-istio`: 将 ServiceRole `ns-access-istio` 绑定到 profile 所有者。
因此 profile 所有者可以通过 Istio（浏览器）访问上述命名空间中的服务。
- 设置命名空间范围内的服务帐户 `editor` 和 `viewer` 供用户在上述命名空间中创建的 pod 使用。
- 资源配额 (自 v1beta1 起)
- 自定义插件 (自 v1beta1 起)

## 支持的平台和先决条件

**GCP**
- 所有用户应该有 IAM 权限 `Kubernetes Engine Cluster Viewer`
  - This is needed in order to get cluster access by `gcloud container clusters get-credentials`
- kubeflow 集群版本在 v0.6.2+
- kubeflow 集群入口设置为 GCP IAP

## 接入控制和资源管理

**[Kubeflow 多租户详细文档](https://www.kubeflow.org/docs/other-guides/multi-user-overview/)**

### manual access management by admin

Cluster admin can manage access management for cluster users:

**To create an isolated namespace `kubeflow-user1` for user `user1@abcd.com`**
- Admin can create a [profile](config/samples/profile_v1beta1_profile.yaml) via kubectl:
```
kubectl create -f /path/to/profile/config
```

**To revoke access to namespace `kubeflow-user1` from user `user1@abcd.com` and delete namespace `kubeflow-user1`**
- Admin can delete profile kubeflow-user1:
```
kubectl delete profile kubeflow-user1
```

### Self-serve kfam UI

用户不用注册即可接入集群 API server，并可不用管理员批准就能使用 kubeflow 集群。


## Profile v1beta1:

**Profile v1beta1 introduced 2 new customizable fields:**

### ResourceQuotaSpec
Profile now support configuring `ResourceQuotaSpec` as part of profile CR.
- `ResourceQuotaSpec` field will accept standard [k8s ResourceQuotaSpec](https://godoc.org/k8s.io/api/core/v1#ResourceQuotaSpec)
- A resource quota will be created in target namespace.
- [Example](config/samples/profile_v1beta1_profile.yaml)

### Plugins
Plugins field is introduced to support customized actions based on k8s cluster's surrounding platform.

Consider adding a plugin when you want to have platform-specific logics like managing resources outside k8s cluster.

Plugin interface is defined as:
```$xslt
type Plugin interface {
	// Called when profile CR is created / updated
	ApplyPlugin(*ProfileReconciler, *profilev1beta1.Profile) error
	// Called when profile CR is deleted, to cleanup any non-k8s resources created via ApplyPlugin
	RevokePlugin(*ProfileReconciler, *profilev1beta1.Profile) error
}
```
Plugin owners have full control over plugin spec struct and implementation.

**Available plugins:**
- [WorkloadIdentity](controllers/plugin_workload_identity.go)
  - Platform: GKE
  - Type: credential binding
  - WorkloadIdentity plugin will bind k8s service account to GCP service account,
  so pods in profile namespace can talk to GCP APIs as GCP service account identity.
- [IAMForServiceAccount](controllers/plugin_iam.go)
  - Platform: EKS
  - Type: credential binding
  - IAM For Service Account plugin will grant k8s service account permission of IAM role,
  so pods in profile namespace can authenticate AWS services as IAM role.
