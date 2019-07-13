# SpringCloud&Docker-Compose

## Why use  docker-compose

+ 每个微服务一般都会部署多个实例。如果每个微服务都要手动启停，那么效率之低、维护量之大可想而知。

## What is docker-compose

+ Compose是一个用于定义和运行多容器Docker应用程序的工具，前身是Fig。

## How do

+ 使用Compose大致有3个步骤
  + 使用Dockerfile定义应用程序环境，以便在任何地方重现该环境
  + 在docker-compose.yml文件中定义组成应用程序的服务，以便各个服务在一个隔离的环境中一起运行
  + 运行docker-compose up命令，启动并运行整个应用程序