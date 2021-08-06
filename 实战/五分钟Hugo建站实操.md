
# 五分钟建站实操指导
> 利用开源的Hugo建站平台，选择一个自己的主题，生成静态内容，发布至服务器上，即建站成功。
> Hugo具有高效，简单，便于维护的特性，是一款强大的建站通用工具。

# 环境准备
+ 操作系统
    + Ubuntu20.04（root权限，便于安装nginx）
+ Web服务器
    + Nginx（安装在Ubuntu）
+ 静态Html部署（利用Hugo进行生成）

# Nginx的安装与配置
> nginx主要作为网站静态资源的容器，安装在操作系统内，监听默认的80端口，可以直接通过服务器ip地址访问。
## Nginx安装
~~~shell script
su root
apt updated
apt install nginx
systemctl status nginx
~~~
> 通过`systemctl status nginx`命令查看启动状态，安装成功后。进行验证。通过浏览器访问服务器地址会出现Nginx的欢迎页面。

## Nginx的配置
> 进入nginx配置目录
~~~shell script
cd /etc/nginx/
ls -al
~~~
~~~
drwxr-xr-x   8 root root 4096 Aug  5 15:02 .
drwxr-xr-x 114 root root 4096 Aug  5 14:30 ..
drwxr-xr-x   2 root root 4096 May 26 01:10 conf.d
-rw-r--r--   1 root root 1077 Feb  4  2019 fastcgi.conf
-rw-r--r--   1 root root 1007 Feb  4  2019 fastcgi_params
-rw-r--r--   1 root root 2837 Feb  4  2019 koi-utf
-rw-r--r--   1 root root 2223 Feb  4  2019 koi-win
-rw-r--r--   1 root root 3957 Feb  4  2019 mime.types
drwxr-xr-x   2 root root 4096 May 26 01:10 modules-available
drwxr-xr-x   2 root root 4096 Aug  5 14:30 modules-enabled
-rw-r--r--   1 root root 1490 Feb  4  2019 nginx.conf
-rw-r--r--   1 root root  180 Feb  4  2019 proxy_params
-rw-r--r--   1 root root  636 Feb  4  2019 scgi_params
drwxr-xr-x   2 root root 4096 Aug  5 15:19 sites-available
drwxr-xr-x   2 root root 4096 Aug  5 15:03 sites-enabled
drwxr-xr-x   2 root root 4096 Aug  5 14:30 snippets
-rw-r--r--   1 root root  664 Feb  4  2019 uwsgi_params
-rw-r--r--   1 root root 3071 Feb  4  2019 win-utf
~~~
> nginx.conf是nginx的配置文件，通过查看内部配置，有`include /etc/nginx/conf.d/*.conf;`和`include /etc/nginx/sites-enabled/*;`
> 两行，表示配置文件包含了/etc/nginx/conf.d/*.conf和/etc/nginx/sites-enabled/*下的所有配置文件。/etc/nginx/sites-enabled/default
> 内包含了/etc/nginx/sites-available，所以我们按照nginx推荐的配置套路，将我们需要的新建站点配置存放在sites-available中。我们可以直接使用它
> 的默认配置。


# Hugo安装
> hugo是基于Go语言进行开发的，所以我们需要先安装golang，然后安装hugo。
## Golang安装
+ 进入https://golang.google.cn/dl/
+ 下载windows版本的安装包
+ 直接进行安装
+ 完成完成后，配置环境变量，GOROOT指定到go的安装目录，GOPATH不需要配置，可以使用默认目录，如果是windows用户，默认路径在c盘的用户目录下，这个目录主要用于存放从仓库下载的依赖包
## go配置
> go语言是目前非常流行的中间件开发语言，拥有python的语法特性，性能居于c语言和java语言之间。它同时也有一个插件管理仓库，但是主要是在国外。所以我们
> 需要配置一下国内的仓库地址。

+ 打开cmd，修改仓库地址，指定到cn代理服务器上
~~~cmd
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
~~~

## Hugo安装（https://gohugo.io/）
> hugo的安装方式有好几种，这里我们推荐用源码安装，因为我们已经安装好了golang包。如果安装了git，可以直接在本地拉去代码
> git clone https://github.com/gohugoio/hugo.git，如果没有git，可以下载源代码的zip包，然后解压到本地目录。

+ 安装hugo，进入源码包目录
    + 执行go install
    + 如果需要支持Sass/SCSS，需要设置go配置，执行CGO_ENABLED=1 go install --tags extended

## 利用hugo创建站点
> 在这里，强烈推荐大家去学习hugo这个建站工具。这里只简单的列出常用的几个操作。
+ hugo new site $sitename   创建站点
+ hugo  源码build，生成静态部署文件存在public中
+ hugo -t $主题目录（主题的引用，可以利用git clone/也可以引用下载好的主题包）
+ hugo server -D（启动服务）

> 这里我们挑选了一个主题`https://github.com/themefisher/meghna-hugo`,meghna-hugo主题下载下来后，其中包含了一个exampleSite,利用它来构建我们的第一个站点。
> 拷贝exampleSite到我们的开发目录，然后利用cmd进入开发目录

~~~cmd
hugo -t $meghna-hugo主题目录地址
hugo （构建源代码，生成静态代码保存在public中）
~~~
> 将public打包上传至服务器的nginx容器中，默认nginx的静态资源存放路径为/var/www/html，将静态代码解压至这里即可，注意nginx配置的目录，index.html的所以路径要与
> 配置路径一致。

> 至此站点搭建完成，后续需要做的就是修改网站内容，非常简单。有兴趣的可以去阅读hugo官方文档，其主要的思维逻辑是，主题中已经定义好了网站的模板（需要注意的是layout的优先级高于主题），而网站的具体内容，通过编辑
> content/下的文件即可，主要就是把每个模块的名称对应上。

> 如果你是新手，肯定不止五分钟，因为你要安装环境。如果你已经安装好了开发环境，那么不需要五分钟，两分钟就可以搞定了。由于这里访问的是国外的网站，可能网络访问不了，所以该墙就墙。


                                          




