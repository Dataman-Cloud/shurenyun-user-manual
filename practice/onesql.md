## OneSQL 入驻数人云  

OneSQL 是一款以 MySQL / PostgreSQL 为基础，融合了超过十年的互联网运维、数据、应用三个层面的架构经验，为电子商务、互联网金融等重要场景进行深层定制的数据库版本，作为真正的 Web Scale SQL，可让 MySQL 在高压情况下始终保持平稳的处理能力，DBA 可在秒杀类场景的瞬间流量下保持淡定，在16颗 Interl E5-2680 CPU 的 PC 上测试简单的 Select，可以压测到每秒36万 QPS。  

OneSQL 的特点在于由互联网行业（如电商、互联网金融等）工作超过十年的资深架构师精心打造，针对高并发、秒杀、关键数据保护等场景进行深入研究和定制，将原本在应用层实现的数据库保护方案下移到 MySQL 的 Server 层，降低了对上层应用开发的要求，使任何公司都可以轻松地拥有与大型互联网企业一样专业数据库技术服务能力，这才是 OneSQL 版本的关键价值所在。

现在，OneSQL 已经入驻数人云，一起来体验一下吧！

<h3 id="step1">1 第一步 制作镜像</h3>

首先，我们需要在 Docker 环境下制作 OneSQL 的 Docker image，并推送至可访问的 Docker Registry。  

· 编写如下配置文件：  

config_mysql.sh

```	
	__mysql_config() {
	# Hack to get MySQL up and running... I need to look into it more.
	echo "Running the mysql_config function."
	cp /etc/onesql/my.cnf /data/onesql/data/
	/usr/local/onesql/scripts/mysql_install_db --defaults-file=/data/onesql/	data/my.cnf --basedir=/usr/local/onesql/ --user=root
	touch /data/onesql/data/mysql_install_db_success
	/usr/local/onesql/bin/mysqld_safe --defaults-file=/data/onesql/data/my.cnf 	--basedir=/usr/local/onesql/ --user=root --socket=/tmp/mysql3306.sock & 
	sleep 10
	}
	__start_mysql() {
	echo "Running the start_mysql function."
	/usr/local/onesql/bin/mysqladmin --socket=/tmp/mysql3306.sock -u root 	password mysqlPassword
	/usr/local/onesql/bin/mysql --socket=/tmp/mysql3306.sock -uroot -	pmysqlPassword -e "CREATE DATABASE testdb"
	/usr/local/onesql/bin/mysql --socket=/tmp/mysql3306.sock -uroot -	pmysqlPassword -e "GRANT ALL PRIVILEGES ON testdb.* TO 'testdb'@'localhost' 	IDENTIFIED BY 'mysqlPassword'; FLUSH PRIVILEGES;"
	/usr/local/onesql/bin/mysql --socket=/tmp/mysql3306.sock -uroot -	pmysqlPassword -e "GRANT ALL PRIVILEGES ON *.* TO 'testdb'@'%' IDENTIFIED BY 	'mysqlPassword' WITH GRANT OPTION; FLUSH PRIVILEGES;"
	/usr/local/onesql/bin/mysql --socket=/tmp/mysql3306.sock -uroot -	pmysqlPassword -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 	'mysqlPassword' WITH GRANT OPTION; FLUSH PRIVILEGES;"
	/usr/local/onesql/bin/mysql --socket=/tmp/mysql3306.sock -uroot -	pmysqlPassword -e "select user, host FROM mysql.user;"
	killall mysqld
	sleep 10
	}
	# Call all functions
	if [ ! -e /data/onesql/data/mysql_install_db_success ]
	then
	    __mysql_config
	    __start_mysql
	fi
```

initDB.sh

```
	#!/bin/bash
	sh /config_mysql.sh
```

my.cnf

```
	[mysqld]
	tcc_control_min_connections=32
	datadir =/data/onesql/data/
	basedir =/usr/local/onesql/ 
	port = 3306
	binlog_format=row
	max_connections=4096
	log_bin=/data/onesql/data/92bin
	innodb_data_file_path=ibdata1:12M:autoextend
	skip_name_resolve
	innodb_log_file_size=1g
	sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
	socket=/tmp/mysql3306.sock
	[mysql]
	socket=/tmp/mysql3306.sock
	[mysqld_safe]
```

run.sh

```
	sh initDB.sh
	/usr/local/onesql/bin/mysqld_safe --defaults-file=/data/onesql/data/my.cnf --datadir=/data/onesql/data/ --user=root --socket=/tmp/mysql3306.sock 
```

· 编写 Dockerfile：  

```  
	FROM centos:centos6  
	MAINTAINER The CentOS Project <cloud-ops@centos.org>  
	RUN yum -y update   
	RUN yum clean all  
	RUN yum -y install wget  
	RUN yum -y install tar  
	RUN yum -y install perl  
	RUN yum -y update; yum clean all  
	RUN yum -y install epel-release; yum clean all  
	RUN yum -y install pwgen supervisor bash-completion psmisc net-tools; yum clean all  
	RUN wget -P /opt http://www.onexsoft.cn/software/onesql-5.6.27-all-rhel6-linux64.tar.gz  
	RUN cd /opt && tar zxvf onesql-5.6.27-all-rhel6-linux64.tar.gz -C /usr/local/  
	RUN ln -s /usr/local/onesql5627 /usr/local/onesql  
	RUN mkdir -p /etc/onesql  
	RUN mkdir -p /data/onesql/data  
	RUN mkdir -p /data/onesql/data/92bin  
	ADD my.cnf /etc/onesql/my.cnf  
	ADD ./run.sh /run.sh  
	ADD ./config_mysql.sh /config_mysql.sh  
	ADD ./supervisord.conf /etc/supervisord.conf  
	ADD ./initDB.sh /initDB.sh  
```  

· 创建并上传 Docker image:  

```
	docker build -t DockerRegistryPath/onesql-server:2.6.17   
	docker push DockerRegistryPath/onesql-server:2.6.17  
```

<h3 id="step1">1 第二步 建立集群</h3>

请参见 [创建/删除集群](../function/create_delete_cluster.md) 来创建您的集群。  

创建集群的实例可以参考[第一个应用-2048](../get-started/2048.md)。

>注意：如果您需要从集群外部来访问服务，则需要配置外网 IP 或可访问域名，集群中要配置外部网关和内部代理，以便对外进行服务暴露。  

<h3 id="step2">2 第三步 发布应用</h3>    
  
接下来，通过数人云创建应用。  

1. 新建 onesql 应用：  
填写应用名称:onesql  
	选择集群：your-cluster  
	添加应用镜像地址：```DockerRegistryPath/onesql-server```  
	填写镜像版本：```2.6.17```   
	选择应用类型：有状态应用  
	添加目录：主机目录：```/var/lib/onesql```，容器目录：```/data/onesql/data```  
	选择容器规格：  CPU：0.5   内存：512 MB  
	容器个数：1  
高级设置：  
	填写应用地址：  端口：3306，类型：对内 TCP，映射端口：3306  
>主机目录与步骤1中创建的目录保持一致，容器目录与 my.cnf 配置文件中的 ```datadir```字段保持一致。  

2. 新建 wordpress 应用：  
填写应用名称:wordpress  
选择集群：your-cluster  
添加应用镜像地址：```wordpress```  
填写镜像版本：latest  
选择应用类型：无状态应用  
选择容器规格：  CPU：0.5   内存：512 MB  
容器个数：1  
高级设置：  
填写应用地址：  端口：80，类型：对外 HTTP， 域名：```your-website```  
参数：  
```WORDPRESS_DB_HOST=proxy-ip:3306```  
```WORDPRESS_DB_USER=root```  
```WORDPRESS_DB_PASSWORD=mysqlPassword```    

等待应用部署完成，访问 wordpress 地址，可以看到如下界面：  
![添加应用](wordpress.png)

恭喜，现在你的 onesql 已经正常运作了！