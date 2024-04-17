## 内容:
针对 kubeadm创建的 cluster运行CIS基准测试工具时，发现了多个必须立即解决的问题。

## 任务:
通过配置修复所有问题并重新启动受影响的组件以确保新的设置生效。

修复针对 API 服务器发现的所有以下违规行为：
1.2.7 Ensure that the --authorization-mode argument is not set to AlwaysAllow FAIL
1.2.8 Ensure that the --authorization-mode argument includes Node FAIL
1.2.9 Ensure that the --authorization-mode argument includes RBAC FAIL
1.2.18 Ensure that the --insecure-bind-address argument is not set FAIL （1.25中这项题目没给出，但最好也检查一下，模拟环境里需要改）
~~1.2.19 Ensure that the --insecure-port argument is set to 0 FAIL ~~（1.25中这项题目没给出，不需要再修改了）

修复针对kubelet发现的所有以下违规行为：
Fix all of the following violations that were found against the kubelet:
4.2.1 Ensure that the anonymous-auth argument is set to false FAIL
4.2.2 Ensure that the --authorization-mode argument is not set to AlwaysAllow FAIL
注意：尽可能使用 Webhook 身份验证/授权。

修复针对etcd发现的所有以下违规行为：
Fix all of the following violations that were found against etcd:
2.2 Ensure that the --client-cert-auth argument is set to true FAIL




## 解题
```shell
kubectl config use-context KSCS00201

mkdir -p /opt/bak1

cp -f /etc/kubernetes/manifests/kube-apiserver.yaml /opt/bak1/
cp -f /etc/kubernetes/manifests/etcd.yaml /opt/bak1
cp -f /var/lib/kubelet/config /opt/bak1

kube-bench master # 检查不安全项
# 修改kube-apiserver.yaml
--authencation-mode=Node,RBAC

# 修改etcd.yaml
--client-cert-auth=true

# 修改kubelet
vim /var/lib/kubelet/config.yaml
修改authentication下的enabled为false，webhook为true，mode为Webhook

kube-bench node # 检查不安全项
systemctl status kubelet

重载kubelet服务
systemctl daemon-reload
systemctl restart kubelet

# 检查，看到所有pod都启动之后再做下一题
kubectl get pods -A
```


## 参考
> https://kubernetes.io/zh/docs/reference/config-api/kubelet-config.v1beta1/
> https://space.bilibili.com/275696381/channel/collectiondetail?sid=1594571

不需要查文档