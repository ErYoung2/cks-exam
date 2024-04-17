## 任务
使用运行时检测工具来检测 Pod tomcat123 单个容器中频发生成和执行的异常进程。
有两种工具可供使用：

sysdig
falco
注： 这些工具只预装在 cluster 的工作节点 node02 上，不在 master 节点。
使用工具至少分析 30 秒 ，使用过滤器检查生成和执行的进程，将事件写到 /opt/KSR00101/incidents/summary 文件中，
其中包含检测的事件， 格式如下：
timestamp , uid/username , processName

## 解题
```shell
kubectl config use-context KSSC00401

ssh node02
sudo -i

# 查看socket信息
crictl info | grep sock

sysdig -l | grep time
sysdig -l | grep uid
sysdig -l | grep proc

sysdig -M 30 -p "%evt.time,%user.uid,%proc.name" --cri /run/containerd/containerd.sock container.name=tomcat123 >> /opt/KSR00101/incidents/summary
sysdig -M 30 -p "%evt.time,%user.name,%proc.name" --cri /run/containerd/containerd.sock container.name=tomcat123 >> /opt/KSR00101/incidents/summary

#启用模块
sysdig-probe-loader

cat /opt/KSR00101/incidents/summary |head
```

需要查文档

