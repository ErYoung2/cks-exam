## 任务
在 namespace istio-system 中获取名为 db1-test 的现有 secret 的内容
将 username 字段存储在名为 /cks/sec/user.txt 的文件中，并将password 字段存储在名为 /cks/sec/pass.txt 的文件中。
注意：你必须创建以上两个文件，他们还不存在。

注意：不要在以下步骤中使用/修改先前创建的文件，如果需要，可以创建新的临时文件。

在 istio-system namespace 中创建一个名为 db2-test 的新 secret ，内容如下：
username : production-instance
password : KvLftKgs4aVH
最后，创建一个新的 Pod ，它可以通过卷访问 secret db2-test ：
Pod 名称 secret-pod
Namespace istio-system
容器名 dev-container
镜像 nginx
卷名 secret-volume
挂载路径 /etc/secret

## 解题

```shell
kubectl config use-context KSCH00701

kubectl -n istio-system get secrets db1-test -o jsonpath={.data.username} | base64 -d > /cks/sec/user.txt
kubectl -n istio-system get secrets db1-test -o jsonpath={.data.password} | base64 -d > /cks/sec/pass.txt

cat /cks/sec/user.txt
cat /cks/sec/pass.txt

kubectl -n istio-system create secret generic db2-test --from-literal username=production-instance --from-literal assword=KvLftKgs4aVH
kubectl -n istio-system get secret
```

然后，创建pod使用此secret
```shell
vim k8s-secret.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod          #pod名字
  namespace: istio-system   #命名空间
spec:
  containers:
  - name: dev-container     #容器名字
    image: nginx            #镜像名字
    volumeMounts:
    - name: secret-volume   #卷名
      mountPath: "/etc/secret"  #挂载路径
  volumes:
  - name: secret-volume     #卷名
    secret:
      secretName: db2-test  #名为 db2-test 的 secret
```

```shell
kubectl -f k8s-secret.yaml create
kubectl -n istio-system get pod
```

## 参考
> https://kubernetes.io/zh/docs/tasks/configmap-secret/managing-secret-using-kubectl/#decoding-secret

> https://kubernetes.io/zh/docs/tasks/configmap-secret/managing-secret-using-kubectl/#create-a-secret

> https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets

需要查文档