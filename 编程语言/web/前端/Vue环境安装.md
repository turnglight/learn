

# WebStorm破解

+ 下载<http://idea.lanyus.com/jar/JetbrainsIdesCrack-3.4-release-enc.jar> 

+ 接下来需要修改文件，使用记事本打开bin文件夹下面的WebStorm.exe.vmoptions 和WebStorm64.exe.vmoptions这两个文件

  + 最后一行添加：       -javaagent:D:/Program Files/JetBrains/WebStorm 2018.3/bin/[JetbrainsIdesCrack-3.4-release-enc.jar](http://idea.lanyus.com/jar/JetbrainsIdesCrack-3.4-release-enc.jar)

+ 两个文件都改好后，再次进入<http://idea.lanyus.com/>，点击获取注册码，注册

  ![img](https://img-blog.csdnimg.cn/20181126103903558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NDk1Nzg2,size_16,color_FFFFFF,t_70)

# Vue环境安装

## 1.安装nodejs，自带npm环境

地址：https://nodejs.org/en/download/，Node.js 历史版本下载地址：https://nodejs.org/dist/ 可自行选择适合自己操作系统的版本。

![img](http://img.php.cn/upload/article/000/054/025/2a4caf3b8ec745259073319c52835842-0.png)

安装过程参见：安装配置

## 2.安装淘宝镜像（提升安装速度）

因国内访问国外链接速度过慢，建议将 npm 的注册表源设置为国内的镜像，可以大幅提升安装速度。

打开cmd直接输入：npm install -g cnpm –registry=https://registry.npm.taobao.org，回车，等待安装...



```
`npm install -g cnpm --registry=https:``//registry.npm.taobao.org`
```

注意=后面不能有空格，否则会报错。

## 3.安装express（选择安装）



```
cnpm install -g express
```

## 4.安装webpack 



```
cnpm install -g webpack
```

## 5.安装vue



```
cnpm install vue
```

## 6.安装vue-cli



```
cnpm install -g vue-cli
```

## 7.项目初始化

在cmd里用cd命令跳转到指定地址，项目初始化命令：

```
vue init webpack vue-projectname
```

projectname--项目文件夹的名称(名字不能用中文)，这个vue-projectname文件夹会自动生成在你跳转到的工作目录中。

![img](http://img.php.cn/upload/article/000/054/025/2a4caf3b8ec745259073319c52835842-1.png)

注意：安装过程中，需要自行输入项目名称，描述，作者等等，即上图中的？后面的答案。

## 8.项目安装部署



```
`cnmp install`
```

![img](http://img.php.cn/upload/article/000/054/025/2a4caf3b8ec745259073319c52835842-2.png)

## 9.测试项目是否安装成功。

```
`cnpm run dev`
```

![img](http://img.php.cn/upload/article/000/054/025/cf39c61c1ca5e4c2a0f2cb342845d20a-3.png)