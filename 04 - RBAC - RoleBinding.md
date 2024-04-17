## 内容
绑定到 Pod 的 ServiceAccount 的 Role 授予过度宽松的权限。完成以下项目以减少权限集。

## 任务
一个名为 web-pod 的现有 Pod 已在 namespace db 中运行。
编辑绑定到 Pod 的 ServiceAccount service-account-web 的现有 Role，仅允许只对 services 类型的资源执行 get 操作。
在 namespace db 中创建一个名为 role-2 ，并仅允许只对 namespaces 类型的资源执行 delete 操作的新 Role。
创建一个名为 role-2-binding 的新 RoleBinding，将新创建的 Role 绑定到 Pod 的 ServiceAccount。

注意：请勿删除现有的 RoleBinding。

## 解题
```shell
kubectl config use-context KSCH00201
kubectl -n db describe rolebindings
kubectl -n db edit role role-1
kubectl -n db describe role role-1
```

# 添加role配置
```yaml
rules:
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
```


```shell
kubectl -n db create role role-2 --verb=delete --resource=namespaces
kubectl -n db create rolebinding role-2-binding --role=role-2 --serviceaccount=db:service-account-web
kubectl -n db describe rolebindings
```


## 参考
> https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/#role-and-clusterole
> https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/#一些命令行工具

需要查文档