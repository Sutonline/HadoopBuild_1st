### 解决的问题:
1. hdfs zkfc不能跟随hdfs启动，需要使用hadoop-daemon.sh start zkfc进行启动，否则两个都是standby
2. hive load数据的问题
	```
	'OPTION SQL_SELECT_LIMIT=DEFAULT'
	```
	在/tmp/root/hive.log中看到是语法错误，在网上一查说是驱动bug，升级驱动就好了。当前用的是5.1.18，升级到5.1.23，问题解决了。

### 待解决的问题:
1. ResourceManager只能起来一个，需要去看看网上的yarn-site.xml
2. 不能通过网页直接看文件内容，只能通过namenode的50070端口看目录及文件
3. mapreduce的编程jar包中的mapper和reducer中的log打印不出来。分别有试syso, log4j, slf4j。看日志的地方在http://hadoop05:8088/cluster,然后选择application，进去选择logs，但是三个文件中都没有我们手动log的信息。
4. 在运行mapreduce运行jar包的时候，hadoop01的namenode有时候不能响应，但是这个节点也有完成的任务。



