


## CICD持续集成概念
+ 持续集成(CI)：持续编译、测试、打包；
+ 持续部署(CD)：代码在任何时刻都是可部署的，并且适配不同的环境自动部署

<img src="https://box.kancloud.cn/2034b60a052b83ec55f7790e798fdf9d_609x480.png">

## 企业级CICD流水线
>1.开发者提交代码到GitLab\
>2.提交代码触发GitLab的WebHook，调用Jenkins构建\
>3.Jenkins跑构建流程,将最终构建的镜像文件推送至Harbor镜像仓库\
>4.通知K8s触发更新服务

<img src="https://box.kancloud.cn/a63d8fc673a8d05787936944999c11d0_705x624.png">

## 技术点
+ git(可参考./Gitlab/Gitlab安装部署)
  > 代码仓库，设置WebHook，来监控代码更新，触发Jenkins，执行构建流程

+ Maven（工程构建）
  > 打包插件，在Jenkins中指定Maven，利用Maven执行打包构建操作

+ Docker（容器服务）
  > 部署服务的轻量级容器，在主机上安装宿主Docker服务，其他Docker服务作为子服务，依赖于宿主Docker进程，但是子服务之间是相互隔离的

+ Jenkins（工程自动化构建）
  + Pipeline流水线构建
  + Maven构建

+ Harbor(主要用于镜像提取加速)
  + 私有Docker镜像仓库
  > Docker默认的镜像仓库是官方的镜像仓库，docker.io\
  > 通过设置**vim /usr/lib/systemd/system/docker.service**，找到ExecStart在后面添加--insecure-registry hub.zlt.com:8888，指定为本地私有镜像仓库

+ Rancher & Kubernates（Docker服务编排管理）
  + Docker容器管理服务
