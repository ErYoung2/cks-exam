## 内容

## 任务
创建一个名为 pod-restriction 的 NetworkPolicy 来限制对在 namespace dev-team 中运行的 Pod products-service 的访问。
只允许以下 Pod 连接到 Pod products-service

namespace qaqa 中的 Pod
位于任何 namespace，带有标签 environment: testing 的 Pod
注意：确保应用 NetworkPolicy。
你可以在 /cks/net/po.yaml 找到一个模板清单文件。

## 解题
```shell
kubectl config use-context KSSH00301

kubectl get ns --show-labels

# 查看 pod 标签（environment: testing）
kubectl get pod -n dev-team --show-labels

# 如果不存在标签，则需要打标签
kubectl label ns qaqa name=qaqa
kubectl label pod products-service environment=testing -n dev-team

vi /cks/net/po.yaml
```

```yaml
metadata:
  name: pod-restriction #修改
  namespace: dev-team #修改
spec:
  podSelector:
  matchLabels:
    environment: testing #根据题目要求的标签修改，这个写的是Pod products-service的标签，也就是使用kubectl get pod -n dev-team --show-labels查出来的pod的标签，这个标签不要和题目里要求的“位于任何namespace，带有标签environment: testing的Pod”这句话里的标签混淆了，两个没有关系，所以可不一样。比如你考试时查出来的POD products-service的标签是name: products，那这里的environment: testing就要换成name: products。
  policyTypes:
  - Ingress #注意，这里只写 - Ingress，不要将 - Egress也复制进来！
  ingress:
  - from: #第一个from
    - namespaceSelector:
      matchLabels:
        name: qaqa #命名空间有name: qaqa标签的
  - from: #第二个from
    - namespaceSelector: {} #修改为这样，所有命名空间
      podSelector: #注意，这个podSelector前面的“-” 要删除，换成空格，空格对齐要对。
        matchLabels:
          environment: testing #有environment: testing标签的Pod，这个地方是根据题目要求“Pods with label environment: testing , in any namespace”，这句话里的pod标签写的。不要和上面spec里的混淆。
```

```shell
# 创建、检查
kubectl apply -f /cks/net/po.yaml
kubectl get networkpolicy -n dev-team
```

## 参考
> https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/#networkpolicy-resource

需要查文档
