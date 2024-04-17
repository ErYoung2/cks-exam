## 内容
APPArmor 已在 cluster 的工作节点node02上被启用。一个 APPArmor 配置文件已存在，但尚未被实施。

## 任务
在 cluster 的工作节点node02上，实施位于 /etc/apparmor.d/nginx_apparmor 的现有APPArmor 配置文件。
编辑位于 /cks/KSSH00401/nginx-deploy.yaml 的现有清单文件以应用 AppArmor 配置文件。
最后，应用清单文件并创建其中指定的 Pod 。

请注意，考试时，考题里已表明APPArmor在工作节点上，所以你需要ssh到开头写的工作节点上。
在模拟环境，你需要ssh到node02这个工作节点。

## 解题
```shell
kubectl config use-context KSSH00401

ssh node02
sudo -i

vi /etc/apparmor.d/etc/apparmor.d/nginx_apparmor
```

```yaml
#include <tunables/global>
#nginx-profile-3 #检查这一行是否存在 存在则注释掉
profile nginx-profile-3 flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}
```

```shell
apparmor_parser -q /etc/apparmor.d/nginx_apparmor
apparmor_status | grep nginx-profile-3
```

**修改pod文件，在terminal上，不在集群节点上！！！**

```shell
vi /cks/KSSH00401/nginx-deploy.yaml
```

```yaml
# 添加注解
annotations:
  container.apparmor.security.beta.kubernetes.io/podx: localhost/nginx-profile-3
```

```shell
# 创建pod
kubectl -f  /cks/KSSH00401/nginx-deploy.yaml create

# 检查
kubectl get pod
kubectl exec podx -- cat /proc/1/attr/current
kubectl exec podx -- touch /tmp/test
```


## 参考
> https://kubernetes.io/zh/docs/tutorials/security/apparmor/#example

需要查文档

