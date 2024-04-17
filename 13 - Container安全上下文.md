## 背景
Container Security Context应在特定namespace中修改Deployment。

## 任务
按照如下要求修改 sec-ns 命名空间里的 Deployment secdep

用ID为 30000 的用户启动容器（设置用户ID为: 30000）
不允许进程获得超出其父进程的特权（禁止allowPrivilegeEscalation）
以只读方式加载容器的根文件系统（对根文件的只读权限）

## 解答
```shell
kubectl config use-context KSMV00102

kubectl -n sec-ns edit deployment secdep
```

```yaml
## Under container
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true

    securityContext:
      runAsUser: 30000
```

## 参考
> https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/

xxxxx