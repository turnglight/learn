## 1.创建maven目录
#创建maven目录
sudo mkdir /usr/maven
#进入maven目录
cd /usr/maven/

## 2.下载maven
+ 下载maven
    + sudo wget https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

+ 解压
    + tar -zxvf apache-maven-3.6.3-bin.tar.gz 

## 3.配置环境变量

#编辑 文件 .bashrc
sudo vim ~/.bashrc
在 .bashrc文件最后面添加添加：

#maven
export M2_HOME=/usr/maven/apache-maven-3.6.1
export CLASSPATH=$CLASSPATH:$M2_HOME/lib
export PATH=$PATH:$M2_HOME/bin

#重新加载资源
source ~/.bashrc

## 4.测试
mvn --version

## 5.配置阿里云镜像
~~~xml
<mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
~~~
