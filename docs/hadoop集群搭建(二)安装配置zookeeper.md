### hadoop集群搭建（二）:安装配置zookeeper

### 准备工作

1. 安装CentOS 64位，在组件的时候选择development server workstation。

2. 检查jdk的版本，自带open jdk 1.7

3. 将虚拟机文件复制为6份.

4. 调整虚拟机的网卡配置和主机名  
	* 在打开虚拟机的时候，会提示选择移动或者复制。选择了复制虚拟机。进去之后发现网卡名称变成Auto eth1或Auto eth2, 需要按照下面的步骤调整。([参考网址](http://hadoopadmintips.blogspot.com/2013/12/issue-in-bringing-up-system-eth0-in.html))
		* 查看/etc/udev/rules.d/70-persistent-net.rules，将原来的重复MAC的eth0删除掉，并且修改eth1然后修改为eth0然后重启.
		* 使用ifconfig查看，这时候网卡应该已经变成了eth0.然后更改/etc/sysconfig/network-script/ifcfg-eth0中的HADDR，注释UUID。
		* 使用service network restart命令重启，重启成功。
	* 修改主机名称，修改文件中的hostname  
		vi /etc/sysconfig/network
5. 配置host文件  
	在/etc/hosts中文件中增加
	```
	192.168.189.201 hadoop01
	192.168.189.202 hadoop02
	192.168.189.203 hadoop03
	192.168.189.204 hadoop04
	192.168.189.205 hadoop05
	192.168.189.206 hadoop06
	```
6. 关闭防火墙
	```
	service iptables stop
	chkconfig iptables off
	```
7. ssh免登陆配置

#### 安装zookeeper

1. 解压文件
	`tar -zxvf zookeeper-3.4.5.tar.gz -C /bigData/`
2. 修改配置  
	`cd /bigData/zookeeper-3.4.5/conf/`  
	`cp zoo_sample.cfg zoo.cfg`  
	修改文件内容:
	```
	dataDir=/bigData/zookeeper-3.4.5/tmp
	#在最后添加：
	server.1=hadoop01:2888:3888
	server.2=hadoop02:2888:3888
	server.3=hadoop03:2888:3888
	```
	在/bigData/zookeeper-3.4.5/conf下建立tmp文件，并且`echo 1 > myid`
3. 将配置好的zookeeper集群文件夹拷贝到其他两台机器hadoop01, hadoop02上，然后修改myid的内容，对应server.N的。
	```
	scp -r zookeeper-3.4.5/ root@hadoop03:/bigData
	scp -r zookeeper-3.4.5/ root@hadoop03:/bigData
	#在hadoop02的机器上
	echo 2 > /bigData/zookeeper-3.4.5/tmp/myid
	#在hadoop03的机器上
	echo 3 > /bigData/zookeeper-3.4.5/tmp/myid
	```

