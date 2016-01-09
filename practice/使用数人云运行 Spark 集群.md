## 使用数人云运行 Spark 集群  

Cassandra 是一个高可靠的大规模分布式存储系统，是一个高度可伸缩的、一致的、分布式的结构化 key-value 存储方案，集 Google BigTable 的数据模型与 Amazon Dynamo 的完全分布式的架构于一身。2007由facebook开发，2009年成为Apache的孵化项目。由于 Cassandra 良好的可扩展性，被 Digg、Twitter 等知名网站所采纳，成为了一种流行的分布式结构化数据存储方案。

接下来，来体验一下用数人云来部署 Cassandra 集群吧。

<h3 id="step1">第一步 制作镜像</h3>

首先，我们需要在 Docker 环境下制作 Spark 的 Docker image，并推送至可访问的 Docker Registry。  

####1. 编写如下配置文件  

mesos-site.xml

```	
	<?xml version="1.0" encoding="UTF-8"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	
	<configuration>
	    
	  <property>
	    <name>mesos.hdfs.namenode.cpus</name>
	    <value>0.25</value>
	  </property>
	
	  <property>
	    <name>mesos.hdfs.datanode.cpus</name>
	    <value>0.25</value>
	  </property>
	    
	  <property>
	    <name>mesos.hdfs.journalnode.cpus</name>
	    <value>0.25</value>
	  </property>
	    
	  <property>
	    <name>mesos.hdfs.executor.cpus</name>
	    <value>0.1</value>
	  </property>
	    
	  <property>
	    <name>mesos.hdfs.data.dir</name>
	    <description>The primary data directory in HDFS</description>
	    <value>/var/lib/hdfs/data</value>
	  </property>
	
	  <property>
	    <name>mesos.hdfs.framework.mnt.path</name>
	    <value>/opt/mesosphere</value>
	    <description>This is the default for all DCOS installs</description>
	  </property>
	
	  <property>
	    <name>mesos.hdfs.state.zk</name>
	    <value>master.mesos:2181</value>
	    <description>See the Mesos DNS config file for explanation for this</description>
	  </property>
	
	  <property>
	    <name>mesos.master.uri</name>
	    <value>zk://master.mesos:2181/mesos</value>
	    <description>See the Mesos DNS config file for explanation for this</description>
	  </property>
	
	  <property>
	    <name>mesos.hdfs.zkfc.ha.zookeeper.quorum</name>
	    <value>master.mesos:2181</value>
	    <description>See the Mesos DNS config file for explanation for this</description>
	  </property>
	
	  <property>
	    <name>mesos.hdfs.mesosdns</name>
	  	<value>true</value>
	    <description>All DCOS installs come with mesos DNS to maintain static configurations</description>
	  </property>
	
	  <property>
	    <name>mesos.hdfs.native-hadoop-binaries</name>
	    <value>true</value>
	    <description>DCOS comes with pre-distributed HDFS binaries in a single-tenant environment</description>
	  </property>
	
	  <property>
	    <name>mesos.native.library</name>
	    <value>/opt/mesosphere/lib/libmesos.so</value>
	  </property>
	  
	  <property>
	    <name>mesos.hdfs.ld-library-path</name>
	    <value>/opt/mesosphere/lib</value>
	  </property>
	</configuration>
```

hdfs-site.xml

```
	<configuration>
	  <property>
	    <name>dfs.ha.automatic-failover.enabled</name>
	    <value>true</value>
	  </property>
	
	  <property>
	    <name>dfs.nameservice.id</name>
	    <value>hdfs</value>
	  </property>
	
	  <property>
	    <name>dfs.nameservices</name>
	    <value>hdfs</value>
	  </property>
	
	  <property>
	    <name>dfs.ha.namenodes.hdfs</name>
	    <value>nn1,nn2</value>
	  </property>
	
	  <property>
	    <name>dfs.namenode.rpc-address.hdfs.nn1</name>
	    <value>namenode1.hdfs.mesos:50071</value>
	  </property>
	
	  <property>
	    <name>dfs.namenode.http-address.hdfs.nn1</name>
	    <value>namenode1.hdfs.mesos:50070</value>
	  </property>
	
	  <property>
	    <name>dfs.namenode.rpc-address.hdfs.nn2</name>
	    <value>namenode2.hdfs.mesos:50071</value>
	  </property>
	
	  <property>
	    <name>dfs.namenode.http-address.hdfs.nn2</name>
	    <value>namenode2.hdfs.mesos:50070</value>
	  </property>
	
	  <property>
    	<name>dfs.client.failover.proxy.provider.hdfs</name>
		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	  </property>
	</configuration>
```  

core-site.xml

```  
	<?xml version="1.0" encoding="UTF-8"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	
	<configuration>
	  <property>
	    <name>fs.default.name</name>
	    <value>hdfs://hdfs</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.hue.hosts</name>
	    <value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.hue.groups</name>
	    <value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.root.hosts</name>
	    <value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.root.groups</name>
	    <value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.httpfs.hosts</name>
	    <value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.httpfs.groups</name>
	    <value>*</value>
	  </property>
	</configuration>
```  

spark-env.sh

```  
  export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:jre/bin/java::")
  export MASTER=mesos://zk://${ZOOKEEPER_ADDRESS}/mesos
  export SPARK_HOME=/opt/spark/dist
  export SPARK_LOCAL_IP=`ifconfig eth0 | awk '/inet addr/{print substr($2,6)}'`
  export SPARK_LOCAL_HOSTNAME=`ifconfig eth0 | awk '/inet addr/{print substr($2,6)}'`
  export LIBPROCESS_IP=`ifconfig eth0 | awk '/inet addr/{print substr($2,6)}'`
```  

spark-default.conf

```  
  spark.mesos.coarse=true
  spark.mesos.executor.home /opt/spark/dist
  spark.mesos.executor.docker.image index.shurenyun.com/mesosphere/spark:1.5.0-hadoop2.6.0
```  

####2. 编写 Dockerfile  

```  
	FROM mesosphere/mesos:0.23.0-1.0.ubuntu1404
	
	# Set environment variables.
	ENV DEBIAN_FRONTEND "noninteractive"
	ENV DEBCONF_NONINTERACTIVE_SEEN "true"
	
	# Upgrade package index and install basic commands.
	RUN apt-get update && \
	    apt-get install -y openjdk-7-jdk curl
	
	ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
	
	ENV MESOS_NATIVE_JAVA_LIBRARY /usr/local/lib/libmesos.so
	
	ADD . /opt/spark/dist
	
	ADD hdfs-site.xml /etc/hadoop/hdfs-site.xml
	ADD core-site.xml /etc/hadoop/core-site.xml
	ADD mesos-site.xml /etc/hadoop/mesos-site.xml
	ADD spark-env.sh /opt/spark/dist/conf/spark-env.sh
	ADD spark-default.conf /opt/spark/dist/conf/spark-default.conf
	
	RUN ln -sf /usr/lib/libmesos.so /usr/lib/libmesos-0.23.1.so
	
	WORKDIR /opt/spark/dist
```  

####3. 创建并上传 Docker image:  

```
	docker build -t your.registry.site/spark:1.5.0-hadoop2.6.0   
	docker push your.registry.site/spark:1.5.0-hadoop2.6.0  
```

* 请把 ```your.registry.site``` 换成你的镜像仓库地址；数人云已将该镜像推送至测试仓库 ```index.shurenyun.com```.

<h3 id="step2">第二步 建立集群</h3>

请参见 [创建/删除集群](../function/create_delete_cluster.md) 来创建您的集群。  

创建集群的实例可以参考[第一个应用-2048](../get-started/2048.md)。 

<h3 id="step3">第三步 发布应用</h3>    
  
登录到集群网络中的一台主机上：
  启动spark driver container
  
  ```
docker run -it --net host -e ZOOKEEPER_ADDRESS=10.3.10.29:2181,10.3.10.63:2181,10.3.10.51:2181 index.shurenyun.com/spark:1.5.0-hadoop2.6.0 bash 
  ```

>注1：CASSANDRA_SEEDS：Cassandra集群的种子节点地址；这个选项可以设置多个值，即Cassandra集群中有多个种子节点，集群中所有的服务器在启动的时候，都将于seed节点进行通信，从而获取集群的相关信息；这里选择3台主机作为 seed 节点；  
>注2：Cassandra 启动需要足够的资源，建议 CPU 数最小为1，内存最低2G；  
>注3：Cassandra 节点间需要通信，所以选择 HOST 模式部署，避免端口隐射导致而节点间无法通信；  
>注4：如果对 Cassandra 集群有大致的规划，可以在选择主机处选择所需数量的主机；应用发布后，可以在所选主机的数量范围内，自由伸缩 Cassandra 节点数量。  

<h3 id="step4">第四步 测试</h3>  

启动Spark shell

```
  bin/spark-shell
```
  
运行demo

```
  sc.parallelize(1 to 1000) count
```

若看到名为 test 的 keyspace 已经添加成功，如下图所示：

![](spark-cluster.png)

恭喜，现在你的 Cassandra 集群已经正常运作了！