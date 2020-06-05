

## 特点
+ 约定优于配置(个人理解为将过去约定俗称的配置套路简化成了一种开发约定，免去开发中的配置，优化了开发流程)
+ 隐藏了组件的复杂性，并提供了声明式、无XML的配置方式
+ 开箱即用（注解开关）、快速启动（基于springboot）
+ 轻量级的组件。SpringCloud为微服务架构提供了非常完整的支持。例如，配置管理、服务发现、断路器、微服务网关等。
+ 选型中立、丰富。例如，SpringCloud的服务发现组件Eurake，可替换为zookeeper或者Consul，Nacos。
+ 灵活。SpringCloud的组成部分是解耦的，开发人员可按灵活挑选技术选型。

## 版本简介
**SpringCloud** 是一个综合项目，它包含了很多子项目。由于子项目也维护着自己的版本号，SpringCloud采用了这种命名方式，从而避免了与子项目的版本混淆。其中，英文单词叫做 **release train** , Angel、Brixton、Camden等都是伦敦地铁站的名称，他们按照字母顺序发行，可将其理解为主版本的演进。SR表示 **Service Release** ,一般表示Bug修复；在SR版本发布之前，会先发布一个Release版本，例如Camden REALEASE。

>+ springcloud版本**发布记录**可详见： **https://github.com/spring-cloud/spring-cloud-release/releases**，从中可以清晰的看到SpringCloud版本发布的时间及先后顺序
>+ SpringCloud版本的**演进计划**: **https://github.com/spring-cloud/spring-cloud-release/milestones**,从中可了解SpringCloud未来版本的演进计划。
