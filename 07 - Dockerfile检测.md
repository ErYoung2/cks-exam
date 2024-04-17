## 任务
分析和编辑给定的Dockerfile /cks/docker/Dockerfile（基于ubuntu:16.04 镜像），
并修复在文件中拥有的突出的安全/最佳实践问题的两个指令。

分析和编辑给定的清单文件 /cks/docker/deployment.yaml ，
并修复在文件中拥有突出的安全/最佳实践问题的两个字段。

注意：请勿添加或删除配置设置；只需修改现有的配置设置让以上两个配置设置都不再有安全/最佳实践问题。

注意：如果您需要非特权用户来执行任何项目，请使用用户ID 65535 的用户 nobody 。

只修改即可，不需要创建。

## 解题
```shell
kubectl config use-context KSSC00301

vim /cks/docker/Dockerfile

FROM ubuntu:16.04
user: nobody

vim /cks/docker/deployment.yaml
# 确保标签一致
privileged: False
runAsUser: 65535
readonlyRootFilesystem: True
```

template里标签跟上面的内容不一致，所以需要将原先的run: couchdb修改为app: couchdb
app: couchdb
注意，这里具体要改成app: couchdb还是其他的标签，要以考试给你的yaml文件的上下文其他标签为准，要与另外两个标签一致，具体见下方截图。）
感谢网友Tse和adams的反馈和纠正。

删除 'SYS_ADMIN' 字段，确保 'privileged': 为False 。
(CKS考试是有多套类似考试环境的，所以有时是删SYS_ADMIN，有时是改'privileged': False）
注意 注意，如果考试时，本来就没有'SYS_ADMIN' 字段，且'privileged':也默认就为False，则直接将直接改成如下。
securityContext:
  privileged: False


## 参考
https://kubernetes.io/zh/docs/concepts/security/pod-security-standards/#restricted

不需要查文档