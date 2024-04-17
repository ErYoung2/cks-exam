## 背景
cluster 上设置了容器镜像扫描器，但尚未完全集成到 cluster 的配置中。
完成后，容器镜像扫描器应扫描并拒绝易受攻击的镜像的使用。

## 任务
注意：你必须在 cluster 的 master 节点上完成整个考题，所有服务和文件都已被准备好并放置在该节点上。

给定一个目录 /etc/kubernetes/epconfig 中不完整的配置，
以及具有 HTTPS 端点 https://image-bouncer-webhook.default.svc:1323/image_policy 的功能性容器镜像扫描器：

启用必要的插件来创建镜像策略
校验控制配置并将其更改为隐式拒绝（implicit deny）
编辑配置以正确指向提供的 HTTPS 端点
最后，通过尝试部署易受攻击的资源 /cks/img/web1.yaml 来测试配置是否有效。

## 解题
```shell
kubectl config use-context KSSH00901

ssh master01
sudo -i

vi /etc/kubernetes/epconfig/admission_configuration.json
```

```json
    "denyTTL": 50,
    "retryBackoff": 500,
    "defaultAllow": false #将true改为false
```

```shell
cp /etc/kubernetes/epconfig/kubeconfig.yml bakyaml/
vi /etc/kubernetes/epconfig/kubeconfig.yml

# 添加webhook server
    certificate-authority: /etc/kubernetes/epconfig/server.crt
    server: https://image-bouncer-webhook.default.svc:1323/image_policy #添加webhook server地址
  name: bouncer_webhook

# 引用ImagePolicyWebhook信息
cp /etc/kubernetes/manifests/kube-apiserver.yaml bakyaml/
vi /etc/kubernetes/manifests/kube-apiserver.yaml

- --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook

- --admission-control-config-file=/etc/kubernetes/epconfig/admission_configuration.json
#在1.25的考试中，默认这行已经添加了，但为了以防万一，模拟环境，没有添加，需要你手动添加。考试时，你先检查，有的话，就不要再重复添加了。重复添加反而会报错！

# 添加volumemount信息
volumeMounts: #在volumeMounts下面增加 #注意，在1.25考试中，蓝色的内容可能已经有了，你只需要添加绿色字体的内容。可以通过红色字，在文件中定位。但模拟环境没有加，需要你全部手动添加。这样是为了练习，万一考试中没有，你也会加。但是如果考试中已添加，你再添加一遍，则会报错，导致api-server启不起来。
- mountPath: /etc/kubernetes/epconfig #建议紧挨着volumeMounts的下方增加
name: epconfig
readOnly: true

# 添加volumes信息
volumes: #在volumes下面增加 #注意，在1.25考试中，蓝色的内容可能已经有了，你只需要检查确认一下是否准确。可以通过红色字，在文件中定位。但模拟环境没有加，需要你全部手动添加。这样是为了练习，万一考试中没有，你也会加。但是如果考试中已添加，你再添加一遍，则会报错，导致api-server启不起来。
- name: epconfig #建议紧挨着volumes的下方增加
hostPath:
path: /etc/kubernetes/epconfig
type: DirectoryOrCreate
#如果你写的是目录，则是DirectoryOrCreate，如果你写的是文件，则是File

# 重启集群并检查
systemctl restart kubelet
kubectl -n kube-system get pod
```
## 参考
> https://kubernetes.io/zh/docs/reference/access-authn-authz/admission-controllers/#如何启用一个准入控制器
> https://kubernetes.io/zh/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook
> https://kubernetes.io/zh/docs/tasks/debug/debug-cluster/audit/#log-后端