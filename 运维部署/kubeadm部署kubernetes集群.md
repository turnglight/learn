
# 环境准备
+ vm
+ centos7.4
+ 准备2-3个主机, 1个master, 1-2个node

# 设置主机名
+ hostnamectl set-name "master"
+ hostnamectl set-name "node1"
+ hostnamectl set-name "node2"

# 初始化
+ 网络配置
> 由于kubernetes的默认仓库在国外，国内无法访问，所以必须要配置国内的镜像仓库，主要是ali的镜像仓库。
> 在连接ALI的仓库地址时，出现了连接超时，相应错误的问题，请检查网络情况，如果网络正常。请检查DNS，极有可能
> 是DNS解析ALI镜像域名失败了，这时可以修改/etc/hosts，去指定IP地址，绕过DNS。（我遇到的坑）
>
    ~~~
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    52.72.232.213    registry-1.docker.io
    120.55.105.209   registry.cn-hangzhou.aliyuncs.com
    47.97.242.13     dockerauth.cn-hangzhou.aliyuncs.com
    192.168.137.4    master
    ~~~
+ docker安装
    + yum install -y yum-utils device-mapper-persistent-data lvm2
    + yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    + yum install docker-ce-18.06.3.ce
    + systemctl enable docker & systemctl start docker
    + docker login --username=13476003847 registry.cn-hangzhou.aliyuncs.com
    + 修改docker加速器
    ~~~
    {
      "registry-mirrors": ["https://kal5ux2i.mirror.aliyuncs.com"],
      "exec-opts": ["native.cgroupdriver=systemd"],
      "insecure-registries": ["registry.cn-hangzhou.aliyuncs.com"]
    }
    ~~~
+ 关闭防火墙
    + systemctl disable firewalld
    + systemctl stop firewalld
+ kubernetes仓库配置
~~~
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
~~~

#  kubeadm安装集群
+ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
+ systemctl enable kubelet & systemctl start kubelet

    + master
        + kubeadm init --kubernetes-version=1.21.3  --apiserver-advertise-address=192.168.137.4   --image-repository registry.aliyuncs.com/google_containers  --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
        + 安装成功后根据提示执行以下命令
            + mkdir -p $HOME/.kube
            + sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
            + sudo chown $(id -u):$(id -g) $HOME/.kube/config
            + export KUBECONFIG=/etc/kubernetes/admin.conf
            + kubectl get nodes   （master安装成功后状态应该为NO READY）
        + master安装成功后，会生成一个加入master的带token的指令
            + kubeadm join 192.168.137.4:6443 --token 7t88ul.x8oovjmrckfk4p88         --discovery-token-ca-cert-hash sha256:8cc005e142a09b9d9e8fb1d49055b210ffa0ecafa03e7b472667232170b275e4
            + 子节点启动了kubelet之后，直接执行上面的命令，即将node加入到master
    + node1 /  node2
        + kubeadm join 192.168.137.4:6443 --token 4tf1d1.u55korkuuveqy1ks         --discovery-token-ca-cert-hash sha256:608114422fc3c08a966f92e2bec0ad6ea3174db4abe6d5947193156ff2aa8250
        
+ master安装网络插件
    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    
+安装验证
    + kubectl get nodes
    + kubectl get pods --all-namespaces

常见问题：
+ kubeadm init：failed to pull image coredns:v1.8.0: error
    + https://my.oschina.net/u/131940/blog/5058471
+ kubernetes node节点加入容器 [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forw
    + sysctl -w net.ipv4.ip_forward=1
