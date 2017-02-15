### hadoop集群搭建（三）:安装hadoop集群

1. 解压hadoop压缩包到bigData  
	`tar -xzvf /root/software/hadoop-2.4.1.tar.gz -C /bigData/`  
2. 配置HDFS  
	配置HADOOP_HOME，因为hadoop2.0所有的配置文件都在$HADOOP_HOME/etc/hadoop目录下.  
	修改文件/etc/profile
	```
	#define HADOOP_HOME
	export HADOOP_HOME=/bigData/hadoop-2.4.1
	export JAVA_HOME=/usr/lib/jvm/jre-1.7.0-openjdk.x86_64
	export PATH=$PATH:${HADOOP_HOME}/bin:${JAVA_HOME}/bin
	```
	`cd $HADOOP_HOME/etc/hadoop`  
	1. 修改hadoop-env.sh，指定JAVA_HOME(虽然在/etc/profile中配置了，但是这里最好再指定一下)。
		```
		export JAVA_HOME=/usr/lib/jvm/jre-1.7.0-openjdk.x86_64
		```	
	2. 修改core-site.xml
		```
		<configuration>
	        <!-- define nameserver to ns1 -->
	        <property>
	                <name>fs.defaultFS</name>
	                <value>hdfs://ns1</value>
	        </property>
	        <!-- define hadoop temporary directory -->
	        <property>
	                <name>hadoop.tmp.dir</name>
	                <value>/bigData/hadoop-2.4.1/tmp</value>
	        </property>
	        <!-- define zookeeper address -->
	        <property>
	                <name>ha.zookeeper.quorum</name>
	                <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
	        </property>
		</configuration>
		```
	3. 修改hdfs-site.xml
		```
		<configuration>
			<property>
				<name>dfs.nameservices</name>
				<value>ns1</value>
			</property>
			<!-- ns1 has two NameNode, nn1, nn2 -->
			<property>
				<name>dfs.ha.namenodes.ns1</name>
				<value>nn1,nn2</value>
			</property>
			<!-- nn1's RPC Address -->
			<property>
				<name>dfs.namenode.rpc-address.ns1.nn1</name>
				<value>hadoop06:9000</value>
			</property>
			<!-- nn1's http address -->
			<property>
				<name>dfs.namenode.http-address.ns1.nn1</name>
				<value>hadoop06:50070</value>
			</property>
			<!-- nn2's rpc address -->
			<property>
				<name>dfs.namenode.rpc-address.ns1.nn2</name>
				<value>hadoop07:9000</value>
			</property>
			<!-- nn2's http address -->
			<property>
				<name>dfs.namenode.http-address.ns1.nn2</name>
				<value>hadoop07:50070</value>
			</property>
			<!-- define NameNode metadata place on JournalNode -->
			<property>
				<name>dfs.namenode.shared.edits.dir</name>
				<value>qjournal://hadoop01:8485;hadoop02:8485;hadoop03:8485/ns1</value>
			</property>
			<!-- define JournalNode metadata store on local disk -->
			<property>
				<name>dfs.journalnode.edits.dir</name>
				<value>/bigData/hadoop-2.4.1/journal</value>
			</property>
			<!-- NameNode failover -->
			<property>
				<name>dfs.ha.automatic-failover.enabled</name>
				<value>true</value>
			</property>
			<!-- FailOver Implementation -->
				<property>
				<name>dfs.client.failover.proxy.provider.ns1</name>
				<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
			</property>
			<!-- policies -->
			<property>
				<name>dfs.ha.fencing.methods</name>
				<value>
					sshfence
					shell(/bin/true)
				</value>
			</property>
			<!-- sshfence need ssh no password login -->
			<property>
				<name>dfs.ha.fencing.ssh.private-key-files</name>
				<value>/home/hadoop/.ssh/id_rsa</value>
			</property>
			<!-- sshfence timeout -->
			<property>
				<name>dfs.ha.fencing.ssh.connect-timeout</name>
				<value>30000</value>
			</property>
		</configuration>
		```
	4. 修改mapred-site.xml
		```
		<configuration>
		   <!-- define map reduce framework use yarn -->
		   <property>
		        <name>mapreduce.framework.name</name>
		        <value>yarn</value>
		   </property>
		
		</configuration>
		```
	5. 修改yarn-site.xml
		```
		<configuration>
			<property>
			   <name>yarn.resourcemanager.ha.enabled</name>
			   <value>true</value>
			</property>
			<!-- define RM's cluster id -->
			<property>
			   <name>yarn.resourcemanager.cluster-id</name>
			   <value>yrc</value>
			</property>
			<!-- define RM's name -->
			<property>
			   <name>yarn.resourcemanager.ha.rm-ids</name>
			   <value>rm1,rm2</value>
			</property>
			<!-- define RM's address -->
			<property>
			   <name>yarn.resourcemanager.hostname.rm1</name>
			   <value>hadoop03</value>
			</property>
			<property>
			   <name>yarn.resourcemanager.hostname.rm2</name>
			   <value>hadoop04</value>
			</property>
			<!-- define zk cluster address -->
			<property>
			   <name>yarn.resourcemanager.zk-address</name>
			   <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
			</property>
			<property>
			   <name>yarn.nodemanager.aux-services</name>
			   <value>mapreduce_shuffle</value>
			</property>
		</configuration>
		```
	6. 修改slaves文件  
		NameNode对应的是DataNode,
		ResourceManager管理NodeManager所以slave是
		```
		hadoop01
		hadoop02
		hadoop03
		```
	7. 将配置好的hadoop拷贝到其他节点
		