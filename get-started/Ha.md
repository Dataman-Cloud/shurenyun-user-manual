# 数人科技BambooHa安装配置文档
    文档信息
    创建人 zhfang
    邮件地址 zhfang@dataman-inc.com
    建立时间 2015年10月21号
    更新时间 2015年10月21号
## 1 环境说明
- Ubuntu 14.04.3 LTS
- 内核版本 3.13.7-031307-generic 
- docker version： 1.8.1
- Storage Driver: aufs
- 与mesos集群内在同一内网

## 2 安装步骤

### 2.1 pull docker 镜像
    docker pull registry.dataman.io/centos7/bamboo-0.24.0-haproxy-1.5.4:omega.v0.2.19
### 2.2 docker参数启动
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

 




####参数说明： 
* MARATHON_ENDPOINT ：marathon地址端口
* BAMBOO_ENDPOINT ：bamboo地址端口
* BAMBOO_ZK_HOST ：zookeeper端口



## 3配置Banboo
###  3.1 将一个域名解析到本台机器
* 例 ：将ha.dataman-inc.net域名解析到10.3.10.127
* 外网访问：http://ha.dataman-inc.net:8000
![ ](./image/bamboo.png)
* 添加Bamboo设置 hdr(host) -i ha.dataman-inc.net
![ ](./image/peizhi.png)
* 外网访问ha.dataman-inc.net就能访问服务了
![ ](./image/2048.png)
