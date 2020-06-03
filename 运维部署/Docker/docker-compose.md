

<a href="https://www.jianshu.com/p/658911a8cff3">Docker:Docker Compose 详解 - 简书</a>


~~~yaml
version: '2' # 表示该 Docker-Compose 文件使用的是 Version 2 file
services:
    eureka_server_1:                                           #服务名
        build: ./springclound/eureka_server_1  #Dockerfile所在目录
        ports:
            - "8761:8761"     #ip,端口映射
        environment:
            - TZ=Asia/Shanghai
        extra_hosts:
            - "eureka-server1:192.168.192.128"
            - "eureka-server2:192.168.192.128"
        volumes:
            - ./springclound/eureka_server_1/logs:/var/log   #挂载容器目录/var/log到宿主机目录/home/runoob/eureka_server1/logs
    eureka_server_2:
        build: ./springclound/eureka_server_2
        ports:
            - "8762:8762"
        environment:
            - TZ=Asia/Shanghai
        extra_hosts:
            - "eureka-server1:192.168.192.128"
            - "eureka-server2:192.168.192.128"
        volumes:
            - ./springclound/eureka_server_2/logs:/var/log
    nginx:
        image: nginx
        ports:
            - "80:80"     #ip,端口映射
        container_name: nginx
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - ./nginx/www:/usr/share/nginx/html  
            - ./nginx/conf/nginx.conf:/etc/nginx/nginx.conf
            - ./nginx/logs:/var/log/nginx
            - /usr/local/static/:/usr/local/static/
    redis:
        #指定镜像
        image: redis
        network_mode: "host"
        container_name: redis
        ports: 
          - 6379:6379
        volumes:
          # 目录映射
           - "./redis/conf:/usr/local/etc/redis"
           - "./redis/data:/data"
        command:
          # 执行的命令
           redis-server
    static: 
       build: ./static
       ports:
            - "8016:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: static
       volumes: 
           - "./static/appsettings.json:/app/appsettings.json"
           - "./static/logs/:/var/app-logs/basic/"
           - "/usr/local/static/:/usr/local/static"
    basic:
       build: ./basic
       ports:
            - "8010:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: basic
       volumes:
           - "./basic/appsettings.json:/app/appsettings.json"
           - "./basic/logs/:/var/app-logs/basic/"
    common:
       build: ./common
       ports:
            - "8013:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: common
       volumes:
           - "./common/appsettings.json:/app/appsettings.json"
           - "./common/logs/:/var/app-logs/common/"

    electest:
       build: ./electest
       ports:
            - "8017:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: electest
       volumes:
           - "./electest/appsettings.json:/app/appsettings.json"
           - "./electest/logs/:/var/app-logs/electest/"
    overhaul:
       build: ./overhaul
       ports:
            - "8014:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: overhaul
       volumes:
           - "./overhaul/appsettings.json:/app/appsettings.json"
           - "./overhaul/logs/:/var/app-logs/overhaul/"
    flaw:
       build: ./flaw
       ports:
            - "8011:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: flaw
       volumes:
           - "./flaw/appsettings.json:/app/appsettings.json"
           - "./flaw/logs/:/var/app-logs/flaw/"
    hidden:
       build: ./hidden
       ports:
            - "8019:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: hidden
       volumes:
           - "./hidden/appsettings.json:/app/appsettings.json"
           - "./hidden/logs/:/var/app-logs/hidden/"
    inspect:
       build: ./inspect
       ports:
            - "8012:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: inspect
       volumes:
           - "./inspect/appsettings.json:/app/appsettings.json"
           - "./inspect/logs/:/var/app-logs/inspect/"
           - "/usr/local/static/upload/:/usr/local/static/upload/"
    prepare:
       build: ./prepare
       ports:
            - "8021:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: prepare
       volumes:
           - "./prepare/appsettings.json:/app/appsettings.json"
           - "./prepare/logs/:/var/app-logs/prepare/"
    protect:
       build: ./protect
       ports:
            - "8015:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: protect
       volumes:
           - "./protect/appsettings.json:/app/appsettings.json"
           - "./protect/logs/:/var/app-logs/protect/"
    trouble:
       build: ./trouble
       ports:
            - "8020:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: trouble
       volumes:
           - "./trouble/appsettings.json:/app/appsettings.json"
           - "./trouble/logs/:/var/app-logs/trouble/"
    emergency:
       build: ./emergency
       ports:
            - "8018:80"
       environment:
            - TZ=Asia/Shanghai
       container_name: emergency
       volumes:
           - "./emergency/appsettings.json:/app/appsettings.json"
           - "./emergency/logs/:/var/app-logs/emergency/"
    risk: 
       build: ./risk
       ports: 
           - "8023:80"
       environment: 
           - TZ=Asia/Shanghai
       container_name: risk
       volumes: 
           - "./risk/appsettings.json:/app/appsettings.json"
           - "./risk/logs/:/var/app-logs/risk/"
    evaluation: 
       build: ./evaluation
       ports: 
           - "8024:80"
       environment: 
           - TZ-Asiz/Shanghai 
       container_name: evaluation
       volumes: 
           - "./evaluation/appsettings.json:/app/appsettings.json"
           - "./evaluation/logs/:/var/app-logs/evaluation/"
    gis: 
       build: ./gis
       ports:
           - "8025:80"
       container_name: gis
       environment:
           - TZ-Asiz/Shanghai
       volumes:  
           - "./gis/appsettings.json:/app/appsettings.json"
           - "./gis/logs/:/var/app-logs/gis/" 

~~~