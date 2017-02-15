### hadoop集群搭建(四)：启动集群及配置hive，sqoop

#### 启动集群

1. 启动zookeeper集群（分别在hadoop01、hadoop02、hadoo03上启动zk）
	```
	cd /bigData/zookeeper-3.4.5/bin/
	./zkServer.sh start
	#查看状态：一个leader，两个follower
	./zkServer.sh status
	```
2. 启动journalnode(为了管理同步服务)
	```
	cd /bigData/hadoop-2.4.1
	sbin/hadoop-daemon.sh start journalnode
	#运行jps命令检验，hadoop01、hadoop02、hadoop03上多了JournalNode进程
	```
3. 格式化HDFS
	```
	#在hadoop06上执行命令:
	hdfs namenode -format
	#格式化后会在根据core-site.xml中的hadoop.tmp.dir配置生成个文件，这里我配置的是/bigData/hadoop-2.4.1/tmp，然后将/bigData/hadoop-2.4.1/tmp拷贝到hadoop07的/bigData/hadoop-2.4.1/下。
	scp -r tmp/ hadoop07:/bigData/hadoop-2.4.1/
	```
4. 格式化ZK(在06节点上执行即可)
	```
	hdfs zkfc -formatZK
    #但是这里格式化成功了，但是zkfc并没有真的配置好，还需要在hdfs-site.xml
	#<property>
   	#	<name>dfs.ha.automatic-failover.enabled</name>
   	#	<value>true</value>
	#</property>
	```
5. 启动hdfs
	```
	sbin/start-dfs.sh
	#这时候在hadoop06, hadoop07上jps会多出来namenode, 但是没有zkfc的进程 需要使用/bigData/hadoop-2.4.1/sbin/hadoop-daemon.sh start zkfc.成功启动了zkfc之后，才会激活一个namenode，否则两个都是standby
	```
6. 启动YARN(在resourceManager上启动,hadoop04启动)
	```
	sbin/start-yarn.sh
	```
7. 到此，hadoop-2.4.1配置完毕，可以统计浏览器访问:
	```
	http://192.168.189.201:50070
	NameNode 'hadoop06:9000' (active)
	http://192.168.189.202:50070
	NameNode 'hadoop07:9000' (standby)
	```
	在namenode或者datanode可以进行上传文件测试。
	mapreduce的测试，可以通过在namenode节点上运行hadoop jar /bigData/hadoop-2.4.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.4.1.jar wordcount /xxx /xxx进行测试

#### 配置hive
 
1. 解压
	`tar -zxvf hive-0.9.0.tar.gz -C /cloud/`  
2. windows上的mysql， 而且虚拟机我们都已经安装了mysql client
3. 配置hive
	```
	cp hive-default.xml.template hive-site.xml 
	#修改hive-site.xml（删除所有内容，只留一个<property></property>）
	#添加如下内容：
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://192.168.189.1:3306/hive?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>root</value>
	  <description>username to use against metastore database</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>123</value>
	  <description>password to use against metastore database</description>
	</property>
	```
4. 安装hive和mysq完成后，将mysql的连接jar包拷贝到$HIVE_HOME/lib目录下
   如果出现没有权限的问题，在mysql授权(在安装mysql的机器上执行)
	```
	mysql -uroot -p
	#(执行下面的语句  *.*:所有库下的所有表   %：任何IP地址或主机都可以连接)
	GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
	FLUSH PRIVILEGES;
	```
5. 可以通过hive命令进行连接，创建表，导入数据进行测试.

#### 配置sqoop

1. 解压tar包
	```
	tar -xzvf sqoop-1.4.4.bin__hadoop-2.0.4-alpha.tar.gz -C /bigData
	```
2. 在添加sqoop到环境变量
	将数据库连接驱动拷贝到$SQOOP_HOME/lib里

3. 可以运行sqoop命令进行测试.