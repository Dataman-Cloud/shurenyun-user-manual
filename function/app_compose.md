####Docker Compose
Docker Compose可以提供 Docker Compose YML的支持，为官方Docker Compose的子集、现已支持如下参数，更多参数后续会陆续兼容，可以从本地文件读取设置。


* Environment

可以设定环境变量，为变量赋值。

    environment: 	
      WORDPRESS_DB_HOST: 192.168.1.12
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: foobar
* links

可以Link到其他的容器。可以直接写应用名(同一个YML内)，或者可以写Link别名(SERVICE:ALIAS)

    links:
     - db
     - db:database
     - redis
Docker Link 会修改您容器内的/etc/hosts文件，就像这样：
	
	 172.17.2.186  db  
	 172.17.2.186  database  
	 172.17.2.186  redis  

* ports

开放端口，可以同时申明主机和容器端口 (HOST:CONTAINER), 也可以只申明容器端口。(会随机选定一个外部端口).

    ports:
     - "3306"
     - "8080:8080"
     - "20048:22"
     - "127.0.0.1:8001:8001"
* image

镜像的地址，如已经在镜像仓库存在、会在主机不存在该镜像的时候，拉取。

    image: ubuntu
    image: orchardup/postgresql
    image: a4bc65fd
* command

覆盖掉默认的cmd命令

    command: bundle exec thin -p 3000



* expose

开放端口但不会在主机上映射。仅仅用于被其他的容器Link。只能保留内部端口

     expose:
     - "3306"
     - "8080"
* volumes

支持Mount存储卷，可以支持指定主机路径和容器路径(HOST:CONTAINER), 还可以包括只读 (HOST:CONTAINER:ro).

    volumes:
     - /var/lib/mysql
     - ./cache:/tmp/cache
     - ~/configs:/etc/configs/:ro


* labels

可以通过Docker Label给容器加一些元数据。

    labels:
      com.example.description: "Accounting webapp"
      com.example.department: "Finance"
      com.example.label-with-empty-value: ""

    labels:
      - "com.example.description=Accounting webapp"
      - "com.example.department=Finance"
      - "com.example.label-with-empty-value"

* net

网络模式. 和命令行参数 --net 相同

    net: "bridge"
    net: "host"


####数人云 Compose
数人云 Compose可以设置应用所占的cpu、内存及实例数，语法与docker Compose类似，可以从本地文件读取。

* cpu: 0.1
  
   		cpu使用量设定，建议0.1~主机最大核心数
  
* mem: 168 
 
 		设定内存使用量、建议16m～主机最大值
 	
* instances: 2
	
		设定应用到实例个数，建议根据资源设定	



