+ 安装redis
    + sudo apt-get install redis-server
+ 查看redis服务状态
    + /etc/init.d/redis-server status
+ 访问redis
    + redis-cli

## 问题
+ 安装后启动失败
~~~
Errors were encountered while processing:
 redis-server
 redis
E: Sub-process /usr/bin/dpkg returned an error code (1)
W: Operation was interrupted before it could finish
root@iZ2zeiudkwjc07o9jfa4fgZ:/var/log/nginx# systemctl status redis-server.service
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; disabled; vendor preset: enabled)
   Active: activating (start) since Fri 2019-01-25 15:15:44 CST; 20s ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 28248 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 2325)
   CGroup: /system.slice/redis-server.service

Jan 25 15:15:44 iZ2zeiudkwjc07o9jfa4fgZ systemd[1]: Starting Advanced key-value store...
Jan 25 15:15:44 iZ2zeiudkwjc07o9jfa4fgZ systemd[1]: redis-server.service: Can't open PID file /var/run/redis/redis-server.pid (yet?) after start: No such file or directory
~~~
 + 查看日志
    + cat /var/log/redis/redis-server.log
 + 解决办法
    + 修改redis配置文件中的 bind 地址；注释 bind 地址或将 bind 地址修改为 0.0.0.0
+ 重新启动
    + systemctl start redis