### Hadoop集群搭建(一):需求

#### 需求:

建立7台虚拟机，安装hadoop, hdfs, hive, mysql, zookeeper, sqoop。

使用zookeeper管理集群，并且运行mapreduce的编程jar包，对日志或者其他文本数据进行分析，只有通过hive进行查询汇总，使用sqoop导入到mysql中。


#### 计划配置:

|主机名|Ip|安装服务|运行的进程|
|-----|--|--------|---------|
|hadoop01|192.168.189.201|hadoop, hive, sqoop, zookeeper|DataNode, NodeManager, JournalNode, QuorumPeerMain|
|hadoop02|192.168.189.202|hadoop, zookeeper|DataNode, NodeManager, JournalNode, QuorumPeerMain|
|hadoop03|192.168.189.203|hadoop, zookeeper|DataNode, NodeManager, JournalNode, QuorumPeerMain|
|hadoop04|192.168.189.204|hadoop|ResourceManager|
|hadoop05|192.168.189.205|hadoop|ResourceManager|
|hadoop06|192.168.189.206|hadoop|NameNode|
|hadoop06|192.168.189.207|hadoop|NameNode|