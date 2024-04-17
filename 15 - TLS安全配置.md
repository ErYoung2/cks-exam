## 任务
通过TLS加强kube-apiserver安全配置，要求

kube-apiserver除了 TLS 1.3 及以上的版本可以使用，其他版本都不允许使用。
密码套件（Cipher suite）为 TLS_AES_128_GCM_SHA256
通过TLS加强ETCD安全配置，要求

密码套件（Cipher suite）为 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

## 解题
```shell
kubectl config user-context KSRS00501

ssh master01
sudo -i

# 备份
mkdir bakyaml
cp /etc/kubernetes/manifests/kube-apiserver.yaml bakyaml/
vim /etc/kubernetes/manifests/kube-apiserver.yaml

# 添加配置项
    - --tls-cipher-suites=TLS_AES_128_GCM_SHA256
    - --tls-min-version=VersionTLS13

# 等待3分钟，待集群策略生效后，再检查kube-apiserver, 确保running
kubectl -n kube-system get pod

# 修改etcd
cp /etc/kubernetes/manifests/etcd.yaml bakyaml/
vim /etc/kubernetes/manifests/etcd.yaml

# 添加配置项
    - --cipher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

# 等待3分钟，再次检查
kubectl get pod -A
kubectl -n kube-system get pod
```

## 参考
> https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver/

需要查文档