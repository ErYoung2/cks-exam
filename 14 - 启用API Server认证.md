## 背景
由 kubeadm 创建的 cluster 的 Kubernetes API 服务器，出于测试目的，
临时配置允许未经身份验证和未经授权的访问，授予匿名用户 cluster-admin 的访问权限.

## 任务
重新配置 cluster 的 Kubernetes APl 服务器，以确保只允许经过身份验证和授权的 REST 请求。
使用授权模式 Node , RBAC 和准入控制器 NodeRestriction 。
删除用户 system:anonymous 的 ClusterRoleBinding 来进行清理。

注意：所有 kubectl 配置环境/文件也被配置使用未经身份验证和未经授权的访问。
你不必更改它，但请注意，一旦完成 cluster 的安全加固， kubectl 的配置将无法工作。
您可以使用位于 cluster 的 master 节点上，cluster 原本的 kubectl 配置文件
/etc/kubernetes/admin.conf ，以确保经过身份验证的授权的请求仍然被允许。

## 解题
```shell
kubectl config use-context KSCF00301

ssh master01
sudo -i
sh /root/b.sh #模拟这道题的环境

# 检查api-server配置文件，是否有以下配置项，将其修改为
- --authorization-mode=AlwaysAllow --> Node, RBAC
- --enable-admission-plugins=AlwaysAdmit --> NodeRestriction

# 重启kubelet
systemctl daemon-reload
systemctl restart kubelet
kubectl get pod -A #过几分钟集群才会恢复正常

# 删除题目要求的角色绑定
kubectl get clusterrolebinding system:anonymous     #检查
kubectl delete clusterrolebinding system:anonymous  #删除
kubectl get clusterrolebinding system:anonymous     #检查

# 验证
kubectl get ClusterRoleBinding system:anonymous --kubeconfig=/etc/kubernetes/admin.conf
kubectl get pods -A --kubeconfig=/etc/kubernetes/admin.conf
less --kubeconfig=/etc/kubernetes/manifests/kube-apiserver.yaml
```

## 参考
> https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/

