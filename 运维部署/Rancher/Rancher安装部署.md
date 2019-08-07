## 介绍
>Rancher是业界唯一完全开源的企业级容器管理平台，为企业用户提供在生产环境中落地使用容器所需的一切功能与组件。Rancher2.0基于Kubernetes构建。使用Rancher，DevOps团队可以轻松测试、部署和管理应用程序，运维团队可以部署、管理和维护一切Kubernetes集群，无论集群运行在何基础设施之上。

## 相关资料
+ rancher官方地址：
https://rancher.com/
+ 中文站点地址如下：
https://www.cnrancher.com/
+ 中文文档地址如下：
https://www.cnrancher.com/docs/rancher/v2.x/cn/overview/quick-start-guide/
+ github源码地址如下：
https://github.com/rancher/rancher

## 安装部署
~~~
sudo docker run -d --restart=unless-stopped -p 9999:9999 -p 443:443 rancher/rancher
~~~

## 浏览器登录
http://120.24.58.111:80 账号：admin  密码：123456
