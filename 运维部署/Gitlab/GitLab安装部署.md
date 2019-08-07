
+ https://www.github.com
+ https://code.aliyun.com
+ 部署私有Gitlab代码仓库

## 部署方式

### 第一种，常规部署
~~~
yum -y install git
git --version
~~~
+ ssh免密配置
  + 第一步：生成Key
    ~~~
    $ ssh-keygen -t rsa -C "you email@email.com" 或者去掉 －t rsa
    ~~~
    >1.接下来会提示你输入一个文件名来保存你的SSH key ，如果不输入直接回车，则会保存在默认的文件里id_rsa.pub\
    >2.然后会提示你输入passphrases 连续输入两次，这在你在网页中添加SSH key的时候会让你输入一次。\
    >**以上提示输入的地方可以为空，敲回车就好。**
    ~~~
    ssh key默认存放地址：/root/.ssh/
    id_rsa.pub：公钥（用于远端服务器）
    id_rsa：私钥
    ~~~
  + 第二步：将公钥（id_rsa.pub 中的内容）添加的git源码服务器\
    将公钥文件 id_rsa.pub 中的内容拷贝到git源码服务器。以阿里云为例。

    ![avatar](http://p1.pstatp.com/large/568000039e9e659fd89e)
    ![avatar](http://p1.pstatp.com/large/568400000dc6a4f42086)
  + 第三步： 验证
    >在存放私钥的服务器终端执行如下命令：\
    >ssh -T git@code.aliyun.com\
    >返回 Welcome to GIT, 用户名! 成功\

### 第二种，Docker部署
+ 下载镜像
  ~~~
  docker pull gitlab/gitlab-ce
  ~~~
+ 运行GitLab容器
  ~~~
  docker run --detach \
           --publish 8443:443 \
           --publish 8480:80 \
           --publish 2222:22 \
           --name gitlab \
           gitlab/gitlab-ce:latest
  ~~~
+ 拷贝配置文件
  ~~~
  docker cp  gitlab:/etc/gitlab/* /opt/gitlab/config/
  ~~~
+ 修改gitlab配置
  ~~~
  vim /opt/gitlab/config/gitlab.rb
  ~~~
  + 修改邮箱配置
    ~~~
    ### Email Settings
    gitlab_rails['gitlab_email_enabled'] = true
    gitlab_rails['gitlab_email_from'] = 'xxx@163.com'
    gitlab_rails['gitlab_email_display_name'] = 'zlt'

    gitlab_rails['smtp_enable'] = true
    gitlab_rails['smtp_address'] = "smtp.163.com"
    gitlab_rails['smtp_port'] = 465
    gitlab_rails['smtp_user_name'] = "xxx@163.com"
    gitlab_rails['smtp_password'] = "客户端授权密码"
    gitlab_rails['smtp_domain'] = "smtp.163.com"
    gitlab_rails['smtp_authentication'] = "login"
    gitlab_rails['smtp_enable_starttls_auto'] = true
    gitlab_rails['smtp_tls'] = true
    ~~~
  + 设置外部url
    external_url 'http://192.168.28.134:8480'
  + 如果ssh端口不是22需要修改端口配置
    gitlab_rails['gitlab_shell_ssh_port'] = 2222

+ 创建启动文件
  ~~~
  vim start.sh
  #!/bin/bash
  GITLAB_DIR=/opt/gitlab
  docker stop gitlab
  docker rm gitlab
  docker run -d \
      -p 8443:443 -p 8480:8480 -p 2222:22 \
      --name gitlab \
      -v ${GITLAB_DIR}/config:/etc/gitlab \
      -v ${GITLAB_DIR}/logs:/var/log/gitlab \
      -v ${GITLAB_DIR}/data:/var/opt/gitlab \
      --privileged=true \
      gitlab/gitlab-ce:latest
  修改HOST_NAME为自己喜欢的名字
  GITLAB_DIR为挂载目录要修改为自己的目录
  ~~~

+ 重新启动容器
  ~~~
  sh start.sh
  ~~~
