## 搭建 Wordpress 个人博客
### 目录
#### [第一步建立集群](#step1)
#### [第二步发布应用](#step2)
#### [第三步服务发现](#step3)

<h3 id="step1">1 第一步建立集群（应用发布环境）</h2>

### 1.1 注册&登录数人云

访问 [www.shurenyun.com](http://www.shurenyun.com) 注册并登录系统。

备注：数人云目前是内测期，如您需要邀请码，请填写免费试用表单申请。  

[免费试用>>](http://form.mikecrm.com/f.php?t=CgBTTT)

### 1.2 准备主机

需要准备至少一台主机，主机可以是可以连接互联网的私有主机，也可以是阿里云、Ucloud、
AWS、Azure、首都在线、华为云等公有云上购买的任意一台云主机。

### 1.3 建立集群

1.3.1 登录账户后，在集群管理中，点击创建群组。（如图所示）

![创建集群1](create-cluster.png)

1.3.2 填写集群名称（sales-2048-demo），选择 1 Master集群，点击完成。

备注：本例是demo，所以选择 1 Master 集群即可。

![创建集群2](create-cluster2.png)

集群已经建立，如下图所示：

![创建集群3](create-cluster3.png)

### 1.4 添加主机

1.4.1 添加主机，如图点击右上角下拉菜单，选择添加主机。

![添加主机](add-host.png)

1.4.2 填写主机名称，并在主机上根据"连接主机"的提示进行操作。   

1.4.3 选择主机类型：
  * 第一台主机为 master 节点，类型为计算节点；
  * 第二台主机选择计算节点和网关节点，用于部署对外的计算服务，该节点需要配置外网 IP 和域名；
  * 第三台主机选择代理节点和数据节点，用于部署有状态的应用，如 mysql、redis 等。

![添加主机](add-host2.png)

（1）安装Docker

	curl -sSL https://get.docker.com/ | sh

（2）安装 Agent

	curl -Ls https://www.shurenyun.com/install.sh | sudo -H sh -s 050f9bb687234f0e9e1e304aa7ddb0ba

执行以上两步后，点击"完成"即成功添加主机。

提示：向同一集群添加的主机应存在于同一网段内，暂不支持跨公网的主机组建集群。

### 1.5 确认集群环境正常

主机添加完成后，检查主机运行是否正常，如图所示：

![添加主机](add-host3.png)

<h3 id="step2">2 第二步发布应用</h2>  
部署 Wordpress 应用，首先需要部署 mysql 数据库，然后部署 Wordpress 服务；我们先从 mysql 开始。  

### 2.1 新建mysql应用

2.1.1 选择"应用管理"中的"创建应用"，如图所示：

![添加应用](add-app.png)

2.1.2 创建应用

填写应用名称:mysql-demo  

添加应用镜像地址：mysql  

填写镜像版本：latest   

选择应用类型：有状态应用  

主机选择：从数据节点中选择一个；有状态应用不能迁移，只能固定在被选择的节点上；另外，有状态应用一次只能部署一个容器；  

容器目录：容器内的挂载目录  

主机目录：主机上的挂载目录  

选择容器规格  

填写暴露端口：3306，类型：对内TCP  

填写环境变量参数：Key:MYSQL_ROOT_PASSWORD  Value:your-password  

如图所示：

![添加应用](add-app2.png)  

### 2.2 新建 wordpress 应用  

2.2.1 选择"应用管理"中的"创建应用"，如图所示：

![添加应用](add-app.png)

2.2.2 创建应用

填写应用名称:wordpress-demo  

添加应用镜像地址：wordpress  

填写镜像版本：latest   

选择应用类型：有状态应用  

选择容器规格  

容器个数：2

填写暴露端口：80，类型：对外标准 HTTP  
注：由于 Wordpress 是 HTTP 应用，并需要对外服务发现，因此选择对外标准 HTTP，会对外暴露 80 端口；同时，需要填写域名：your-site；  

填写环境变量参数：  
Key:WORDPRESS_DB_HOST  Value:10.3.10.63:3306  
Key:WORDPRESS_DB_USER  Value:root
Key:WORDPRESS_DB_PASSWORD  Value:your-password

如图所示：

![添加应用](add-app2.png)  

### 2.3 确认应用正常运行

回到应有管理中，即可看到应用已正常运行。

![添加应用](add-app3.png)  

打开浏览器，访问地址：http://your-site，看到如下页面，则说明wordpress 应用已经成功运行。


<h3 id="step3">3 第三步服务发现，let's play !</h2>

### 3.1 环境说明
- 注意：请将Bamboo安装到Master主机中
- Ubuntu 14.04.3 LTS
- 内核版本 3.13.7-031307-generic
- docker version： 1.8.1
- Storage Driver: aufs
- 与mesos集群内在同一内网

### 3.2 Bamboo和Haproxy的安装步骤

### 3.2.1 通过 docker 镜像安装Bamboo和Haproxy
    docker pull registry.dataman.io/centos7/bamboo-0.24.0-haproxy-1.5.4:omega.v0.2.19

### 3.2.2 docker参数启动
    vi bamboo.sh
    docker run -d \
        -e "MARATHON_ENDPOINT"="http://marathonip1:8080,http://marathonip2:8080,http://marathonip3:8080" \
        -e "BAMBOO_ENDPOINT"="http://bambooip:8000" \
        -e "BAMBOO_ZK_HOST"="zk1:2181,zk2:2181,zk3:2181" \
        --name bamboo-haproxy --net host --privileged --restart always \
        registry.dataman.io/centos7/bamboo-0.24.0-haproxy-1.5.4:omega.v0.2.19

    example

    docker run -d \
        -e "MARATHON_ENDPOINT"="http://10.3.10.103:8080,http://10.3.10.104:8080,http://10.3.10.105:8080" \
        -e "BAMBOO_ENDPOINT"="http://10.3.10.127:8000" \
        -e "BAMBOO_ZK_HOST"="10.3.10.103:2181,10.3.10.104:2181,10.3.10.105:2181" \
        --name bamboo-haproxy --net host --privileged --restart always \
        registry.dataman.io/centos7/bamboo-0.24.0-haproxy-1.5.4:omega.v0.2.19

    sh bamboo.sh


##### 参数说明：
* MARATHON_ENDPOINT ：marathon地址端口
* BAMBOO_ENDPOINT ：bamboo地址端口
* BAMBOO_ZK_HOST ：zookeeper端口

### 3.3 配置Bamboo

####  3.3.1 将一个域名解析到本台机器

* 例 ：将ha.dataman-inc.net域名解析到10.3.10.127

* 外网访问：http://ha.dataman-inc.net:8000

![ ](bamboo.png)

* 添加Bamboo设置 hdr(host) -i ha.dataman-inc.net

![ ](peizhi.png)

* 外网访问ha.dataman-inc.net就能访问服务了

![ ](2048.png)

备注：如对Bamboo详细设置感兴趣，请查看 [Bamboo官方信息>>](https://github.com/QubitProducts/bamboo)
