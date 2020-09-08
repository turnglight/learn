

> 说明：

+ 1.通过设置gitlab-runner实例，gitlab自动再服务器拉取代码。
+ 2.在项目下找到gitlab-ci.yml文件，执行流程化脚本
+ 3.已java程序为例
    + 3.1 mvn安装编译
    + 3.2 将编译好的jar拷贝到指定目录，在该目录下存在一个Dockerfile文件，用于构建镜像
    + 3.3 stop容器，删除旧的镜像，利用docker命令构建新镜像
    + 3.4 启动容器

## 准备
1. 安装gitlab，gitlab-runner
2. 对项目创建runner实例

## JAVA项目的容器化自动部署
> 1. 安装相关依赖

+ 安装maven
+ 安装docker

> 2. docker环境相关准备
    + Docker主服务
    + 项目Dockerfile

> 3. 编写gitlab-ci.yml, 改文件是gitrunner&CICD的流程化脚本

> gitlab-runner命令
+ stages: 阶段集合
+ stage: 具体阶段
+ before_script: 前序脚本
+ script: 主脚本
+ after_script: 后续脚本

~~~yml
stages:
    - build
    - deploy

job_build:
    stage: build
    before_script:    
        - export MAVEN_HOME=/usr/maven/apache-maven-3.6.3
    script:
        - pwd
        - cd ntms-cloud-parent
        - $MAVEN_HOME/bin/mvn install
        - cd ../
        - $MAVEN_HOME/bin/mvn clean compile package
        - docker build -t ntms_cloud_app -f AppDockerfile .
        
job_deploy:
    stage: deploy
    script:
        - docker stop ntms_cloud_app
        - docker rm ntms_cloud_app
        - docker run -d --name ntms_cloud_app -p 18888:18888 -v /root/upload/:/root/upload/ -v /var/java/log:/var/log -e TZ=Asia/Shanghai ntms_cloud_app
~~~

> 说明：mvn命令如果识别不了，需要设置before_script:export MAVEN_HOME=/usr/maven/apache-maven-3.6.3, 而后再下面进行引用，如$MAVEN_HOME/bin/mvn install



