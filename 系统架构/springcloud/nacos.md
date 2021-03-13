
+ 安装nacos服务
+ 启动nacos服务
>  查看启动日志，登录nacos控制台，Console: http://10.255.1.6:8848/nacos/index.html，默认账号密码nacos

## nacos config

+ 添加nacos-config依赖包
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>${alibaba.nacos}</version>
</dependency>
~~~
+ 添加外部化配置
    + 登录控制台，在配置列表中添加dataid的配置,dataid需要遵循一定的格式规范${${prefix}-${profile}.${file-extension:properties}
        + prefix: 默认为spring.application.name，也可以配置spring.cloud.nacos.config.prefix属性
        + profile：后台服务配置的环境属性spring.profile.active
        + file-extension:properties: 配置文件扩展名
        + 例如：demo-dev.yml/demo-dev.properties
    + 配置bootstrap.yml/bootstrap.properties
        ~~~xml
            spring:
                application:
                    name: nacos-config
                cloud:
                    nacos:
                    config:
                        server-addr: 10.255.1.6:8848
                        file-extension: yaml
                profiles:
                    active: dev
        ~~~


## nacos discovery
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>${alibaba.nacos}</version>
</dependency>
~~~

