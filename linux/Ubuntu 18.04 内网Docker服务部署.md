
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

