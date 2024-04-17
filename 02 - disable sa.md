## 内容
您组织的安全策略包括：

ServiceAccount不得自动挂载API凭据
ServiceAccount名称必须以“-sa”结尾
清单文件 /cks/sa/pod1.yaml 中指定的Pod由于ServiceAccount指定错误而无法调度。

## 任务
在现有namespace qa 中创建一个名为 backend-sa 的新ServiceAccount，
确保此ServiceAccount不自动挂载API凭据。
使用 /cks/sa/pod1.yaml 中的清单文件来创建一个Pod。
最后，清理 namespace qa 中任何 未使用的 ServiceAccount。

在现有namespace qa 中创建一个名为 backend-sa 的新ServiceAccount，
确保此ServiceAccount不自动挂载API凭据。
使用 /cks/sa/pod1.yaml 中的清单文件来创建一个Pod。
最后，清理 namespace qa 中任何 未使用的 ServiceAccount。

## 解题
```shell
kubectl config use-context KSCH00301


kubectl create sa backend-sa -n qa
kubectl edit sa backend-sa -n qa
加autoMountServiceAccountToken: false

kubectl -n qa get sa
kubectl -n qa get pod -o yaml|grep -i serviceAccount
kubectl -n qa delete sa test01
```

## 参考
> https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-service-account/#使用默认的服务账户访问-api-服务器

需要查文档