## 任务
在cluster中启用审计日志。为此，请启用日志后端，并确保：

日志存储在 /var/log/kubernetes/audit-logs.txt
日志文件能保留 10 天
最多保留 2 个旧审计日志文件
/etc/kubernetes/logpolicy/sample-policy.yaml 提供了基本策略。它仅指定不记录的内容。

注意：基本策略位于 cluster 的 master 节点上。

编辑和扩展基本策略以记录：

RequestResponse 级别的 persistentvolumes 更改
namespace front-apps 中 configmaps 更改的请求体
Metadata 级别的所有 namespace 中的 ConfigMap 和 Secret 的更改
此外，添加一个全方位的规则以在 Metadata 级别记录所有其他请求。

注意：不要忘记应用修改后的策略。

## 解题
日志审计这一题需要自己调整的内容还是挺多的，因此要非常仔细，建议修改前备份一下原始的环境，要不然修改错了就会导致集群崩溃。

```shell
kubectl config use-context KSCH00601
ssh master01
sudo -i
cp /etc/kubernetes/logpolicy/sample-policy.yaml /opt/bak1/
vim /etc/kubernetes/logpolicy/sample-policy.yaml
```

```yaml
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["persistentvolumes"] #根据题目要求修改，比如题目要求的是namespaces。

  - level: Request
    resources:
    - group: ""
      resources: ["configmaps"] #根据题目要求修改，比如题目要求的是persistentvolumes或者pods。
    namespaces: ["front-apps"]

  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets", "configmaps"]

  - level: Metadata
    omitStages:
      - "RequestReceived"
```

```shell
vim /etc/kubernetes/manifests/kube-apiserver.yaml
# 在spec-container-command下添加参数
spec:
  container:
    - commands:
      - kube-apiserver
      - --audit-log-path=/var/log/kubernetes/audit-logs.txt
      - --audit-log-maxage=10
      - --audit-log-maxbackup=2
      - --aufit-policy-file=/etc/kubernetes/logpolicy/sample-policy.yaml

systemctl daemon-reload
systemctl restart kubelet

# 检查
kubectl get pods -A
tail -100f /var/log/kubernetes/audit-logs.txt
```


## 参考
> https://kubernetes.io/zh/docs/tasks/debug/debug-cluster/audit/

需要查文档