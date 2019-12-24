# Linux内网环境下的相关规则设置

> 用root权限执行一下修改操作
~~~markdown
> 修改了ssh的22端口为10022
vim /etc/ssh/ssh_config
# 追加
port 10022
先不要删除22端口，重启ssh，systemctl restart ssh
确保10022可以远程连接后，可删除22
~~~


弱口令
root   密码修改为    Whfjad2019!@#$
设置grub的密码为     Whfjad2019!@#$

## 1.检查关键敏感文件的权限是否恰当
~~~
chmod 400 /etc/shadow
chmod 444 /etc/passwd
chmod 644 /etc/group
~~~


## 2.检查用户口令最短长度   违规
+ 需要添加系统检查密码规则的插件
+ 离线安装检查密码插件包以及依赖包
~~~
dpkg -i libcrack2_2.9.2-1build2_amd64.deb
dpkg -i  libpwquality-common_1.3.0-0ubuntu1_all.deb
dpkg -i libpwquality1_1.3.0-0ubuntu1_amd64.deb
dpkg -i libpam-pwquality_1.3.0-0ubuntu1_amd64.deb
如果是在线安装直接运行
apt -y install libpam-pwquality
~~~

+ 修改最小密码长度
root@ubuntu: vim /etc/pam.d/common-password
~~~Markdown
# 修改第25行，添加retry=3 minlen=8，中间需要添加空格
# line 25: set 8 for minimum password length
password        requisite                       pam_pwquality.so retry=3 minlen=8
~~~

## 3.检查用户口令最长有效时间
root@ubuntu: vim /etc/login.defs
~~~Markdown
# 修改最大有效天数180天
# line 160: set 60 for Password Expiration
PASS_MAX_DAYS 180

# 修改最小有效天数30天
# line 161: set 1 for Minimum number of days available
PASS_MIN_DAYS 30

# 修改警告天数30天
# line 162: set 7 for number of days for warnings
PASS_WARN_AGE 30
~~~

## 4.禁用ctrl+alt+delete组合键
~~~
systemctl mask ctrl-alt-del.target
systemctl daemon-reload
~~~


## 5.修改登录日志的留存时间为16周
Linux操作系统默认登录日志留存4周，根据网络安全法要求，系统日志必须留存6个月以上（26周）。
~~~
vim /etc/logrotate.conf

# keep 4 weeks worth of backlogs
rotate 4
~~~


## 6.Grub口令保护设置

~~~markdown
> 1.生成grub密钥
grub-mkpasswd-pbkdf2
> 2.设置启动用户，并配置密钥
vim /etc/grub.d/40_custom
> 设置启动用户为root,密钥为上一步生成的密钥
set superusers="root"
password_pbkdf2 root grub.pbkdf2.sha512.10000.BCxxxxxxxxxxxxx45572D3B4E40500CE
> 3.更新grub
update-grub
~~~

## 7.禁用IP路由源（及其它相关内核参数配置）
~~~Markdown
> 1.查看IP路由源配置
sysctl -a | grep accept_source_route
> 2.修改/etc/sysctl.conf,将accept_source_route=1的参数直接拷贝导sysctl.conf，并将值修改为0，保存即可
~~~

## 8.关闭rabbitmq控制台
rabbitmq-plugins enable rabbitmq_management

## 9.设置redis密码
在主机上增加一个/root/redis/conf/redis.conf的文件
~~~
requirepass whfjad@redis
~~~
在docker-compose.yml文件中挂载redis的配置文件，并设置容器中执行命令redis-server /usr/local/etc/redis/redis.conf
~~~
redis:
        #指定镜像
        image: redis
        network_mode: "host"
        container_name: redis
        ports:
          - 6379:6379
        volumes:
          # 目录映射
           - "./redis/conf/redis.conf:/usr/local/etc/redis/redis.conf"
           - "./redis/data:/data"
        command:
          # 执行的命令
           redis-server /usr/local/etc/redis/redis.conf

~~~

## 10.禁止访问swagger-ui
需要在nginx中禁止访问接口服务中的index.html，直接跳转到nginx服务的80端口下
~~~
upstream api {
    server 192.168.137.128:80
}

location ~ /.*/index.html.* {
　　auth_basic "接口文档";
　　proxy_pass http://api;
　　proxy_redirect off;
　　proxy_set_header Host host;
    proxy_set_header X−Real−IP $remote_addr;
　　proxy_set_header X-Forwarded-For       $proxy_add_x_forwarded_for;
    proxy_set_header Remote_addr $remote_addr;
}
~~~

## 11.禁止访问Eurake的管理页面
在nginx中拦截8761和8762端口访问

## 12.开启数据库的日志记录
+ mysql5.7\
vim /etc/mysql/mysql.conf.d/mysqld.cnf
~~~
general_log = 1
general_log_file = /var/log/mysql.log
~~~
执行systemctl restart mysql, 重启mysql。即可查看/var/log/mysql.log日志。
+ mysql5.6\
同样是在/etc/mysql/目录下，但是下面的子目录可能细微差异，修改内容不变
+ 查看修改是否开启  
mysql> show global variables like '%log%';

## 13.修改rabbitmq的默认端口
linux 修改rabbitmq 默认端口
vi sbin/rabbitmq-env
找5672
里面会有+10000  +2000的另外端口