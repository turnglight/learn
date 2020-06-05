# springboot可执行应用JAR简刨

+ 官方文档中，开发人员构建可执行jar，需要添加spring-boot-maven-plugin到pom文件中。

> 在springboot项目目录下，执行：`mvn pakcage`,打包完成后，在target目录中可以得到 ***-0.0.1-SNAPSHOT.jar

## 1.可执行jar资源结构
+ 解压生成的jar包

    ├─BOOT-INF
    │  │  ├─classes
    │  │  └─lib
    │  ├─META-INF
    │  │  └─maven
    │  └─org

    + BOOT-INF/classes目录存放应用编译后的class文件
    + BOOT-INF/lib目录存放应用依赖的JAR包
    + META-INF/目录存放应用相关的元信息，如MANIFEST.MF文件
    + org/目录存放SpringBoot相关的class文件
 + 另一个***-0.0.1-SNAPSHOT.jar.original包实际上被repackage成的上面的JAR包，original包中未引入第三方依赖包，所以它的体积很小，以K记单位。

## 2.JAR包执行------(spring-boot-loader)
+ 当springboot项目使用java -jar命令启动时，其命令本身对jar文件是否来行springboot插件打包并不感知。换言之，它是标准的可执行JAR文件。
+ 按照标准java -jar命令规范，命令引导的具体启动类必须配置在MANIFEST.MF资源的Main-class属性中。
+ 查看解压的jar包目录，/META-INF/MANIFEST.MF，发现其中的Main-class属性设置为`org.springframework.boot.loader.JarLauncher`
+ `org.springframework.boot.loader.JarLauncher`文件存放在org/springframework/boot/loader/目录下
+ 并且项目的引导类定义在Start-Class属性中，该属性并非JAVA平台标准META-INF/MANIFEST.MF属性

以上这些信息在spring官方文档中有描述
~~~
3.1. Launcher Manifest
You need to specify an appropriate Launcher as the Main-Class attribute of META-INF/MANIFEST.MF. The actual class that you want to launch (that is, the class that contains a main method) should be specified in the Start-Class attribute.

The following example shows a typical MANIFEST.MF for an executable jar file:

Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.mycompany.project.MyApplication
For a war file, it would be as follows:

Main-Class: org.springframework.boot.loader.WarLauncher
Start-Class: com.mycompany.project.MyApplication
~~~

`总结`:java -jar命令通过MANIFEST.MF执行的Main-class文件，启动用org.springframework.boot.JarLauncher, JarLauncher的main实现通过读取Start-Class属性，来启动SpringBootApplication类。详细启动过程，可以读取`org.springframework.boot.loader.JarLaunher`源码。
