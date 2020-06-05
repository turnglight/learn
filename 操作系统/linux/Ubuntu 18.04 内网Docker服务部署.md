# Ubuntu18.04内网Docker服务部署
## 1.准备工作
> 在内网环境下，无法连接Internet网络，在安装Docker镜像的过程中，无法直接下载Dockerfile文件中依赖的镜像，需要在已经部署好的环境好，打包完整镜像，拷贝至内网系统中，进行安装
+ 安装Ubuntu18.04的系统，并做好网络配置
+ 准备Docker软件安装包
+ 准备Docker-compose的软件包
+ 准备需要安装的Docker镜像（内网环境下，没有Internet网络，无法下载依赖镜像，需要自行准备好完整镜像）

## 2.打包完整镜像
> 在连接Internet的系统中打包完整镜像
~~~Markdown
> 打包mysql
docker save -o mysql-v5.6.tar  mysql:5.6
> 打包nginx
docker save -o /docker/nginx.tar  nginx
> 打包redis
docker save -o /docker/redis  redis
~~~

## 3.Docker宿主服务安装
> 已经下载好的docker-19.03.2.tgz的Docker安装包
+ tar  -xzvf  docker-19.03.2.tgz   （解压）
+ sudo cp docker/* /usr/bin        （拷贝至/usr/bin）
+ sudo dockerd &                   （启动Docker服务）

## 4.docker-compose工具安装
> docker-compose是一个编排docker服务的管理工具，便于微服务的维护管理
+ 下载docker-compose镜像（docker-compose-Linux-x86_64）
+ 存放在/usr/local/bin,文件命名为docker-compose
+ sudo chmod +x /usr/local/bin/docker-compose

## 5.加载镜像文件
> 将第2步中已经打包好的镜像文件，拷贝至该内网系统中
+ sudo docker load -i redis.tar
+ sudo docker load -i nginx.tar
+ sudo docker load -i mysql-v5.6.tar
> 加载完成后，可以通过`docker images`命令查看到已存在的镜像文件

## 6.运行docker服务
> 利用docker-compose编排好docker服务，下面给一个例子
~~~yml
version: "2"
services:
    nginx:                                           #服务名
        image: nginx
        ports:
            - "80:80"     #ip,端口映射
        container_name: nginx_steeltoe
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - ./build/nginx/www:/usr/share/nginx/html  
            - ./build/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
            - ./build/nginx/logs:/var/log/nginx
            - /usr/local/static/:/usr/local/static/
    mysql:
        image: mysql:5.6
        restart: always  #unless-stopped
        command: --default-authentication-plugin=mysql_native_password #这行代码解决无法访问的问题
        container_name: mysql_test
        volumes:
            - "./build/mysql/data:/var/lib/mysql"
            - "./build/mysql/conf:/etc/mysql/conf.d"
            - "./build/mysql/logs:/logs"
        command: [
            #'--character-set-server=utf8mb4',
            #'--collation-server=utf8mb4_unicode_ci',
            '--default-time-zone=+8:00'
        ]
        environment:
            # 环境变量
            - MYSQL_ROOT_PASSWORD=roottest
            #- MYSQL_DATABASE=ntms
            #- MYSQL_USER=ntms
            #- MYSQL_PASSWORD=ntmstest
        ports:
            - 3111:3306
    redis:
        # 指定镜像
        image: redis
        network_mode: "host"
        ports:
          # 端口映射
          - 6379:6379
        volumes:
          # 目录映射
          - "./build/redis/conf:/usr/local/etc/redis"
          - "./build/redis/data:/data"
        command:
          # 执行的命令
          redis-server
~~~
+ `image`: 对应系统中已经存在的镜像名称
+ `ports`: 端口的映射，将宿主系统的80端口映射至docker'子系统'中的80端口，此处需要去查看docker的原理，通俗一点来讲，每一个docker服务是一个从主系统中切分出来的一个子系统，这是Linux虚拟化的几个核心点之一，要想访问docker服务，必须通过宿主机的端口映射来访问
+ `volumes`: 数据卷，例如数据库的服务镜像，在启动时，需要挂载对应的数据，此时的数据实际上是在宿主机的某一个目录下面，通过目录映射，在启动mysql的镜像服务时，将主机的目录挂载到子系统中，从而实现数据库的数据访问，这也为我们提供了方便，当我们想要修改服务中的某一些配置时，也可以通过这种挂载的方式来实现，而不是登录到子系统中去操作docker镜像内的文件，因为这样会带来风险。

## 7.启动服务
> 为了便于管理，我们编辑了一个docker-compose.yml文件，将它防止在/docker目录下面，同时也可以将其它一些docker服务的相关文件放置在该目录下,下面以启动nginx为例子
+ cd /docker
+ docker-compose up -d nginx   （构建nginx服务）
+ docker-compose start nginx        （启动）
    + docker-compose restart nginx  （重启）
    + docker-compose up -d --force-recreate nginx (强制销毁并重新构建)
=======
# Ubuntu18.04内网Docker服务部署
> 在内网环境下，无法连接外部网络\
> 1.安装docker/docker-compose\
> 2.安装软件镜像，镜像文件无法通过网络下载，只能在已经部署好的环境中拷贝镜像到内网环境中进行部署


## 1.安装docker/docker-compose
+ 下载Docker安装包(docker-19.03.2.tgz)
+ 安装docker
    + tar -zxvf docker-19.03.2.tgz
    + cp docker/* /usr/bin
    + dockerd &
+ 安装docker-compose编排插件
+ 下载docker-compose
    + cp docker-compose /usr/local/bin
    + chmod +x /usr/local/bin/docker-compose

## 2.复制镜像
+ docker save -o /docker/nginx.tar nginx（将现有的nginx镜像进行打包，保存到/docker/nginx.tar文件中）

## 3.安装镜像
> 将复制的镜像文件拷贝到本地
+ docker load -i nginx.tar
