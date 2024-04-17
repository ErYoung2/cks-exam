## 内容
该 cluster 使用 containerd 作为 CRI 运行时。containerd 的默认运行时处理程序是 runc 。
containerd 已准备好支持额外的运行时处理程序 runsc (gVisor)。

## 任务
使用名为 runsc 的现有运行时处理程序，创建一个名为 untrusted 的 RuntimeClass。
更新 namespace server 中的所有 Pod 以在 gVisor 上运行。
您可以在 /cks/gVisor/rc.yaml 中找到一个模版清单。


## 解题
```shell
kubectl config use-context KSMV00301

vim /cks/gVisor/rc.yaml
```

```yaml
apiVersion: node.k8s.io/v1   ##将apiVersion: node.k8s.io/v1beta1修改为apiVersion: node.k8s.io/v1。这个在1.25正式考试的时候，是对的，不需要修改的。
kind: RuntimeClass
metadata:
  name: untrusted # 用来引用 RuntimeClass 的名字，RuntimeClass 是一个集群层面的资源
handler: runsc    # 对应的 CRI 配置的名称
```

```shell
kubectl -f /cks/gVisor/rc.yaml create
kubectl get RuntimeClass
```

将命名空间为server下的Pod引用RuntimeClass。
考试时，3个Deployment下有3个Pod，修改3个deployment即可。

```shell
kubectl -n server get deployment

kubectl -n server edit deployments busybox-run
kubectl -n server edit deployments nginx-host
kubectl -n server edit deployments run-test

```

```yaml
spec: #找到这个spec，注意在deployment里是有两个单独行的spec的，要找第二个，也就是下面有containers这个字段的spec。
  runtimeClassName: untrusted #添加这一行，注意空格对齐，保存会报错，忽略即可。
  containers:
  - image: nginx:1.9
    imagePullPolicy: IfNotPresent
    name: run-test
```
## 参考
> https://kubernetes.io/zh-cn/docs/concepts/containers/runtime-class/#2-创建相应的-runtimeclass-资源

需要查文档