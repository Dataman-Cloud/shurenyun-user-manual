###RUN SPARK ON 数人云

###单机版
####
 - Mesos运行在host上
 - Spark driver 和 executor运行docker上

#####1.搭建Mesos集群环境
  [详情](https://www.shurenyun.com) ，登录数人云，登录控制台后，通过集群管理创建自己的集群
#####2.运行Spark demo
  登录到master节点主机上：
  启动spark driver container
  ```shell
  docker run -it --net host mesosphere/spark:1.5.0-hadoop2.6.0 bash 
  ```
  修改spark配置文件
  ```shell
  cd $SPARK_HOME
  vi conf/spark-defaults.conf
  
  ########################
  spark.mesos.coarse=true
  spark.mesos.executor.home /opt/spark/dist
  spark.mesos.executor.docker.image mesosphere/spark:1.5.0-hadoop2.6.0
  ########################

  vi conf/spark-env.sh

  #####################
  export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:jre/bin/java::")
  export MASTER=mesos://$MESOS_MASTER_IP:5050    #将MESOS_MASTER_IP替换为ip地址，多master请使用zk连接串
  export SPARK_HOME=/opt/spark/dist
  export SPARK_LOCAL_IP=`ifconfig eth0 | awk '/inet addr/{print substr($2,6)}'`
  export SPARK_LOCAL_HOSTNAME=`ifconfig eth0 | awk '/inet addr/{print substr($2,6)}'`
  #####################

  ```
  启动Spark shell
  ```shell
  bin/spark-shell
  ```
  运行demo
  ```scala
  sc.parallelize(1 to 1000) count
  ```

