# Docker

## Docker产生原理

~~~
LXC和AUFS是Docker运行的基础。
~~~

### LXC

> LXC是**整个Docker运行的基础**。
>
> 众所周知，CPU、内存、I/O、网络等都称之为系统资源，而Linux内核有一套机制来管理其所拥有的这些资源，这套机制的核心被称之为CGroups和Namespaces。
>
> CGroups可以限制、记录、调整进程组所使用的物理资源。
>
> NameSpaces则是另外一个重要的资源隔离机制。Namespaces将进程、进程组、IPC、网络、内存等资源都变得不再是全局性资源，而是将这些资源从内核层面属于某个特定的Namespace。在不同的Namespace之间，这些资源是互相透明、不可见的。
>
> 而LXC就是基于Linux内核通过调用CGroups和Namespaces，来实现容器轻量级虚拟化的一项技术，与此同时，LXC也是一组面向Linux内核容器的用户态API接口。通过通过LXC提供的资源限制和隔离功能，可以创建一套完整并且相互隔离的虚拟应用运行环境。

### AUFS

> AUFS的功能简单说就是可以**将分布在不同地方的目录挂载到同一个虚拟文件系统中**。
> AUFS文件系统：
> 	LInux启动过程，首先加载bootfs(Boot File System)目录。这个目录里面包括Bootloader和Kerenl。Bootloader用来加载启动Kernel。当Kerenl成功加载到内存中后，bootfs就会释放掉，kerenl随之就开始加载rootfs。
> 	rootfs（Root File System）包含的是Linux系统中标准的/dev、/proc、/bin、/etc等文件。因为rootfs是后续kerenl的启动基础，对于kerenl来说异常重要，因此此时kerenl将rootfs加锁---设置readonly。在只读权限下，kerenl进行一系列的检查操作。当kerenl确认rootfs包含的文件正确无误后，将readonly改为readwrite（可读可写），以后用户就可以按照正确的权限对这些目录进行操作了。
> 	AUFS登场。当Docker利用**LXC虚拟化**出来一个容器之后，就相当于购买了一台裸机，有内存、CPU、硬盘，但是没有操作系统。Docker参考Linux的启动过程，将一个readonly权限的bootfs挂载到容器文件系统中，然后通过AUFS，再讲readonly权限的rootfs添加到bootfs之上，当rootfs检查完毕之后，再将用户所要使用的文件内容挂载到rootfs之上，同样是readonly权限。每次挂载一个FS文件层，并且每层之间只会挂载增量。
> 	这样文件层就是堆栈式文件系统中所保存的数据。将不同的文件层挂载到同一个文件系统中的文件系统，就是联合文件系统；而AUFS就是用来管理、使用这些文件层的文件系统，因此也称之为高级多层次统一文件系统（Advanced Multi Layered Unification Filesystem）。
> 	但是每个FS层都是readonly权限，那么容器内部如何向这些文件写入数据呢？其实当Docker利用AUFS加载完最高一层之后，会在最上面再添加一个FS层，而这个层就是readwrite权限。容器内部的应用，对当前文件系统所有的写操作（包括删除）都会保存在这个FS层当中，而当容器想Docker发出commit命令后，**Docker会将这个FS层中的数据作为单独一个文件层保存AUFS之中**。
> 	而一个**镜像**就可以理解为：**特定FS层的集合**。所以可以看出镜像的层次关系，处理下层的image是上层image的父类，而没有父类image的就是baseimage.因此需要从image启动container时，Docker会一次加载baseimage和父类image，而用户所有的操作就都保存在最高层的readwrite的layer中。
> 	通过将镜像“分隔”为AUFS的文件层，使得所有容器都可以共享文件层，且不会发生写冲突。但在Docker中，所有的镜像都是只读的，所有的镜像也都不保存用户信息，智慧用于新建和符合。而对于容器而言，其所看到的所有文件都是可读写的，只不过所有的写操作都被保存在最上层的文件层当中。
> 	Docker正是通过AUFS的这些特性，解决了容器初始化和写时复制问题，所有Docker选择AUFS作为其第二个核心组件。



## Dockerfile

> Dockerfile是一个可以读懂的脚本文件。这个脚本文件记录着用户创建镜像过程当中需要执行的所有命令

> AUFS文件系统会将不同的文件层进行**叠加**从而形成一个完成的文件系统。这里在**Dockerfile**中执行命令后形成的临时文件层就是AUFS所使用的文件层。
>
> 这一组文件层，就称之为**镜像**。
>
> 因此，**镜像**指的就是一组特定的文件层，Docker中镜像的构建过程，就是Docker执行**Dockerfile**中所定义命令而形成这组文件的层的过程。

### Dockerfile语法

~~~cmd
# 在Dockerfile语法中，#表示单行注释
command argument argument ..
# Dockerfile语法，就是命令+参数+参数
# 例如，我们想要在镜像构建过程中输出一个hello world
RUN echo "Hello Docker!"
# RUN是Dockerfile内置命令，表示在镜像中执行后面的命令
~~~
