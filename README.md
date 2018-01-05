# Nutanix Kubernetes Volume Plugin

使用Nutanix后台的ABS存储服务，通过iSCSI协议链接，在k8s worker节点上使用nutanix提供的provisioner容器。

here I come.

## 需求

* 本文以CentOs/RHEL 7 为例子
* 在所有的节点上安装和运行 yml install iscsi-initlator-utils -y； 并且启用这个服务
* 在所有节点上加载ntnx/nutanixabs-provisioner镜像文件

## 配置步骤

1. Login Master node：运行 kuberctl create -f serviceaccount.yaml
2. kuberctl get sa
3. kuberctl  create -f clusterrole.yaml
4. kuberctl get clusterrole |grep nutanix
5. kuberctl create -f clusterrolebinding.yaml
6. kuberctl get clusterrolebinding |grep nuta
7. kuberctl create -f deploymebt.yaml， 所有的节点应该可以正常下载或者已经具有ntnx/nutanixabs-provisioner镜像文件
8. kuberctl get po -o wide
9. Go to work nodes, docker ps , 会看到ntnx/nutanixabs-provisioner的容器启动，用docker logs 命令可以查看它的运行情况
10. Got to Master nodes, kuberctl create -f silver.yaml
11. kuberctl create -f gold.yaml
12. kuberctl get sc , 会看到创建的存储类型
13. kuberctl create -f silver-claim.yaml 
14. kuberctl describe pvc
15. Go to worker node, 用docker logs 查看tnx/nutanixabs-provisioner容器的日志，可以看到有volume生成并绑定了
16. Go to Master node. 用 kuberctl get pvc  命令应该可以看到silver的卷处于绑定状态
17. Go to Prisume or acli 可以看到有新的存储vg被动态生成了，id和k8s节点上的一致。在界面里找到并记录，external iscs initlator name
18. 在所有worker节点上修改 /etc/iscsi/initiatorname.iscs 的内容为，InitiatorName=iqn.11111-33.com.nutanix:k8s-worker 每个worker都修改这个配置文件，重启iscis服务 systemctl restart iscsid
19. Go to master node, kuberctl create -f rc_nginx.yaml 部署测试nginx应用
20. kuberctl get pod 应该就可以看到所有nginx正在使用这个卷
