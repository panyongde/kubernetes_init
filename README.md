# kubernetes_init

install docker, kubernetes and initialize kubernetes cluster by kubeadm

在天朝特殊网络环境下，加速安装docker,kubernetes并创建kubernetes集群

## 环境安装

### 安装 VirtaulBox

[下载地址](https://www.virtualbox.org/wiki/Downloads)

### 下载 Ubuntu Server

[下载地址](http://mirrors.163.com/ubuntu-releases/16.04/)

### 安装三台服务器

安装三台服务器，分别为 `master`、`node1`、`node2`

为了方便使用，三台虚拟机请使用`网桥网卡`的连接方式，

然后在路由器中修改DHCP服务，为虚拟网卡物理地址分配固定ip

为服务器安装ssh服务，方便宿主机连接虚拟机，这样方便在终端复制粘贴

```shell
# 在虚拟机终端里执行
sudo su
apt-get install openssh-server
/etc/init.d/ssh start
```



#### 安装Master节点

```shell
# 在宿主机终端登录虚拟机
# 切换到超级账户
sudo su
cd
# 安装 master (在master机器上进行)
git clone https://github.com/panyongde/kubernetes_init.git
cd kubernetes_init
swapoff -a
# 修改 install.sh, 一定要(至少)改前面几个变量， 否则并不会执行成功
./install.sh pre   					# 安装 docker、kube* 等基础工具
./install.sh kubernetes-master   	# 利用kubeadm安装master节点
./install.sh post 					# 安装网络组件
```

如果在执行 ./install.sh kubernetes-master 因网络问题卡住或者遇到错误，是不能再执行一遍的，需要进行重置

```shell
# 设置docker镜像国内源
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://6f410ddf.m.daocloud.io
systemctl restart docker.service

# 重置
kubeadm reset
# 然后再安装master节点
./install.sh kubernetes-master
```



#### 安装Node节点

```shell
# 在宿主机终端登录虚拟机
# 切换到超级账户
sudo su
cd

# 从安装master输出获取相关参数，然后安装node (在node机器上进行)
git clone https://github.com/panyongde/kubernetes_init.git
cd kubernetes_init
swapoff -a
# 修改 install.sh, 一定要(至少)改前面几个变量， 否则并不会执行成功
./install.sh pre   					# 安装 docker、kube* 等基础工具
./install.sh kubernetes-node   		# 利用kubeadm把node添加进cluster
```



## 注意事项
1. 对照着[官方文档](https://kubernetes.io/docs/setup/independent/install-kubeadm/)使用，本脚本几乎按照官方步骤执行，只是修改了各种下载地址，方便国内特殊网络环境使用
2. 根据自己需求修改`install.sh`
3. 为了简单和直观，`install.sh`脚本没有做过多容错处理，当执行错误时(例如忘记修改相关参数)，并不好重新执行一遍，此时应该拷贝出脚本相关内容，手动执行

## 博客链接
https://www.jianshu.com/p/0e54aa7a20cf
讲了一些细节，仅供参考
