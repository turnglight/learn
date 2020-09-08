# Ubuntu 16.04/18.04 搭建KVM安装windows server2012

## How to Install and Configure KVM on Ubuntu 18.04 LTS Server
引用：<a href="https://www.linuxtechi.com/install-configure-kvm-ubuntu-18-04-server/">https://www.linuxtechi.com/install-configure-kvm-ubuntu-18-04-server/</a>

*KVM允许我们安装和运行多个虚拟机（Windows和Linux）。我们可以通过virt-manager图形用户界面或virt-install和virsh cli命令创建和管理基于KVM的虚拟机。*

### 验证系统是否支持虚拟化

+ 在egrep命令下面执行以验证您的系统是否支持硬件虚拟化\
> egrep -c '(vmx|svm)' /proc/cpuinfo\
> sudo apt install cpu-checker
+ 使用以下命令安装“ kvm-ok ”实用程序，该实用程序用于确定您的服务器是否能够运行硬件加速的KVM虚拟机\
> sudo kvm-ok\

### 安装KVM需要依赖的软件包
> sudo apt update\
+ 运行以下apt命令以安装KVM及其依赖项
> sudo apt install qemu qemu-kvm libvirt-bin  bridge-utils  virt-manager
+ 每当我们在Ubuntu 18.04 Server中安装qemu和libvirtd软件包时，它将自动启动并启用libvirtd服务；如果未启动并启用libvirtd服务，则在命令下运行，

### 启动并启用libvirtd服务
> sudo service libvirtd start\
> sudo update-rc.d libvirtd enable\
+ 现在，使用以下命令验证libvirtd服务的状态，
> service libvirtd status\


### 为KVM虚拟机配置网桥
+ 需要网络桥接器来访问KVM虚拟机管理程序或主机之外的基于KVM的虚拟机。
    +  Ubuntu 18.04的网络配置
        > sudo vim /etc/netplan/50-cloud-init.yaml\
        ~~~yaml
        network:
            ethernets:
                eno1:
                dhcp4: no
                dhcp6: no
            bridges:
                bro:
                interfaces: [eno1]
                dhcp4: no
                addresses: [192.168.0.155/24]
                gateway4: 192.168.0.1
                nameservers:
                    addresses: [192.168.0.1]
            version: 2

        ~~~
        > sudo netplan apply\
        > sudo netplan --debug  apply\
        > sudo networkctl status -a （验证网桥状态）
    
    + Ubuntu 16.04的网络配置
        > sudo vim /etc/network/interfaces
        ~~~source /etc/network/interfaces.d/*
        auto lo
        iface lo inet loopback
        auto bro
        iface bro inet static
        address 192.168.0.165
        netmask 255.255.255.0
        gateway 192.168.0.1
        network 192.168.0.0
        broadcast 192.168.0.255
        dns-nameservers 10.8.8.8

        bridge_ports eno1
        bridge_stop off
        bridge_fd 0
        bridge_maxwait 0
        ~~~
        > systemctl restart networking.service (执行后需要重启)

### 创建虚拟机

+ 准备一个集成了virtio驱动的windows server2012的ISO系统镜像，下载地址: https://code.zheteng.win/os/VirtIO/cn_windows_server_2012_r2_x64_virtio.iso
+ 下载vnc客户端，在windows中通过vnc登录windows的系统图新界面，其中要指定端口和密码
+ 创建虚拟机磁盘空间： qemu-img create -f qcow2 -o preallocation=metadata /opt/windows_server_2012.raw 200G

#### Bridge桥接模式
> virt-install \
> --name=windows_server_2012 \
> --ram 8096 --vcpus=16\
> --disk path=/opt/windows_server_2012.raw,bus=virtio \
> --cdrom /home/gly/cn_windows_server_2012_r2_x64_virtio.iso  \
> --graphics vnc,listen=0.0.0.0,port=5921,password=123456 \
> --network bridge=bro,model=virtio --hvm --virt-type kvm  \
> --force --autostart
+ 5921是vnc连接虚拟机的端口

#### Nat模式
> 此处默认开启的vnc端口为5921，在安全级别较高的内网环境下，建议修改默认端口，5921->25921
+ 在内网环境中，通常IP是有限的，虚拟机与宿主机共用一个IP地址，这是需要以NAT的网络连接模式安装虚拟机,设置network=default，因为默认模式就是NAT
> virt-install \
> --name=windows_server_2012 \
> --ram 8096 --vcpus=16 \
> --disk path=/opt/windows_server_2012.raw,bus=virtio \
> --cdrom /home/gly/cn_windows_server_2012_r2_x64_virtio.iso  \
> --graphics vnc,listen=0.0.0.0,port=25921,password=123456 \
> --network network=default,model=virtio --hvm --virt-type kvm  \
> --force --autostart
+ 此时，宿主机的有一个虚拟网卡，IP地址为192.168.122.1, 通过VNC客户端进入虚拟机，windows虚拟机的IP地址为，192.168.122.X，虚拟机可以直接ping通宿主机的主机IP，而宿主机不能ping通虚拟机的IP，此时需要关闭虚拟机的防火墙，就可以通了。
+ 网络通了之后


# 修改VNC端口
+ virsh list –all命令查看现有虚拟机
+ cd /etc/libvirt/qemu/windows_server_2012.xml
+ vim /etc/libvirt/qemu/windows_server_2012.xml
    + 配置VNC的配置文件, port 是手动指定端口号, autoport 是主机自己分配端口号。 这里，我们使用自己指定的端口，所以 autoport 设置为 ‘no’。
    + 默认为5921，修改端口
+ 使命令生效
    + virsh define /etc/libvirt/qemu/windows_server_2012.xml
+ 命令强行关机
    + virsh destroy centos-lnmp
+ 开机即可生效
    virsh start centos-lnmp


### 常用命令
+ virsh list --all  （列出所有虚拟机）
+ virsh undefine win_10 (卸载虚拟机)
+ virsh destroy win_10 (删除虚拟机)
+ systemctl stop libvirtd （停止libvirtd服务）
+ qemu-img create -f qcow2 -o preallocation=metadata /opt/windows_server_2012.raw 200G (创建磁盘空间)

virsh是kvm虚拟机常用的管理工具，以下是一些常用的命令。
1、查看在运行的虚拟机
virsh list
2、查看创建的所有虚拟机
virsh list --all
3、启动虚拟机
virsh start win10
win10是虚拟机的domain名称，下同。
4、挂起虚拟机
virsh suspend win10
5、恢复被挂起的虚拟机
virsh resume win10
6、开机启动虚拟机，即在虚拟机服务（libvirt）启动的时候，就启动虚拟机。
virsh autostart win10
7、关闭开机启动虚拟机
virsh auto start --disable win10
8、关闭虚拟机（需要ACPID服务的支持，ACPI是Advanced Configuration and PowerInterface缩写，高级配置和电源管理接口）
virsh shutdown win10
9、强制关闭虚拟机，这种方式是从virsh list列表中将虚拟机删除，仍然可以start起来
virsh destory win10
10、启动默认的网络
virsh net-start default
11、自启动默认网络
virsh net-autostart default
12、彻底删除虚拟机步骤
1）关闭虚拟机：virsh destroy win10
2）删除定义：virsh undefine win10
3）删除虚拟机文件（在/var/lib/libvirtd/下的相关文件）