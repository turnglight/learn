##安装
说明：当使用私有镜像后，远程镜像是否可用，可以设置多个镜像仓库。Docker默认的是官方的镜像仓库。
+ [下载地址](https://github.com/goharbor/harbor)
+ 解压
  ~~~markdown
  tar -xvf harbor-offline-installer-v1.7.1.tgz
  ~~~
+ 修改配置
  + /harbor.yml
    ~~~markdown
    hostname=?
    port=?
    ~~~
  + /docker-compose.yml
    ~~~markdown
    修改proxy:port属性
    80:80修改为8888:80      8888应喻harbor.yml的port保持一致
    ~~~
+ 开始安装
  ~~~markdown
  sudo ./prepare
  sudo ./install
  ~~~
+ 浏览器访问
  > http://[ip]:8888
+ 服务器docker登录
  > sudo docker login ip:port\
  >
  > 遇到的疑难**问题**\
  > **harbor Error response from daemon: https://hub.zlt.com:8888/v2/_ping: http:server gave HTTP response to HTTPS client** \
  >  **原因**：因为Docker从1.3.X之后，与docker registry交互默认使用的是https，然而此处搭建的私有仓库只提供http服务，所以当与私有仓库交互时就会报上面的错误。为了解决这个问题需要在启动docker server时增加启动参数为默认使用http访问。\
  >  **解决**：修改docker启动配置文件\
  > **vim /usr/lib/systemd/system/docker.service**\
  > 找到ExecStart在后面添加**--insecure-registry hub.zlt.com:8888**\
  > **重启**\
  > systemctl daemon-reload \
  > systemctl restart docker

+ 服务器docker私服重新登录
  > sudo docker login ip:port\
  > Username: admin
  > Password: Harbor12345
  > Login Succeeded
+ 可以开始对进行进行tag，push操作
