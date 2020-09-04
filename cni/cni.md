## CNI
Container Network Interface (CNI) 最早是由CoreOS发起的容器网络规范，是Kubernetes网络插件的基础。其基本思想为：Container Runtime在创建容器时，先创建好network namespace，然后调用CNI插件为这个netns配置网络，其后再启动容器内的进程。现已加入CNCF，成为CNCF主推的网络模型。

CNI插件包括两部分：
* CNI Plugin负责给容器配置网络，它包括两个基本的接口
  * 配置网络: AddNetwork(net NetworkConfig, rt RuntimeConf) (types.Result, error)
  * 清理网络: DelNetwork(net NetworkConfig, rt RuntimeConf) error
* IPAM Plugin负责给容器分配IP地址，主要实现包括host-local和dhcp。

## 过程
Kubernetes Pod 中的其他容器都是Pod所属pause容器的网络，创建过程为：
1. kubelet 先创建pause容器生成network namespace
2. 调用网络CNI driver
3. CNI driver 根据配置调用具体的cni 插件
4. cni 插件给pause 容器配置网络
5. pod 中其他的容器都使用 pause 容器的网络

  
