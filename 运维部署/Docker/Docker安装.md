## 环境安装

+ 阿里云centos7

### docker安装

~~~markdown
> sudo yum install -y yum-utils   device-mapper-persistent-data   lvm2
> sudo yum-config-manager     --add-repo https://download.docker.com/linux/centos/docker-ce.repo
> sudo yum install docker-ce docker-ce-cli containerd.io
> sudo systemctl start docker
> sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
> sudo chmod +x /usr/local/bin/docker-compose
> docker-compose --version
~~~
