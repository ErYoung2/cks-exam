## 任务
使用 Trivy 开源容器扫描器检测 namespace kamino 中 具有严重漏洞的镜像 的 Pod。

查找具有 High 或 Critical 严重性漏洞的镜像，并删除使用这些镜像的 Pod 。

注意：Trivy 仅安装在 cluster 的 master 节点上，
在工作节点上不可使用。
你必须切换到 cluster 的 master 节点才能使用 Trivy


## 解题
```shell
kubectl config use-context KSSC00401

# 查看namespace下的pod和image
kubectl get pod -n kamino -o=custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image"

# 提取镜像名并进行扫描
kubectl describe pod -n kamino | grep -i image: |awk '{print $2}' | sort -u

for i in `kubectl describe pod -n kamino | grep -i image: |awk '{print $2}' | sort -u`; do trivy -q image -s HIGH,CRITICAL $i | grep -iEB3 "HIGH:|CRITICAL:" ; done

# 删除包含漏洞的pod
kubectl -n kamino delete pod tri111 tri222 tri333
```

## 参考
> https://kubernetes.io/zh-cn/docs/reference/kubectl/cheatsheet/#格式化输出

需要查文档