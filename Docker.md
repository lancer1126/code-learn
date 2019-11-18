# Docker

## 概念

Docker 能够自动执行重复性任务，例如搭建和配置开发环境，从而解放了开发人员以便他们专注在真正重要的事情上：构建杰出的软件。
用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

**docker主机(host)** : 安装了Docker程序的机器（docker直接安装在操作系统之上<br>
**docker客户端（Client）**：链接docker主机进行操作<br>
**docker仓库（Repository）**：用来保存各种打包好的软件镜像
**docker镜像（Images）**：软件打包好的镜像，放在docker仓库中
**docker容器（Container）**：镜像启动后的实例称为一个容器，容器是独立运行的一个或一组应用

## 安装Docker

官方教程<br>
<https://docs.docker.com/install/linux/docker-ce/ubuntu/> <br>

1. **SET UP THE REPOSITORY**
   1. 	  $ sudo apt-get update
   2. 	  $ sudo apt-get install \
		    apt-transport-https \
		    ca-certificates \
		    curl \
		    gnupg-agent \
		    software-properties-common
    3. 	   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    4.     $ sudo apt-key fingerprint 0EBFCD88
   
2. **INSTALL DOCKER ENGINE - COMMUNITY**
   1.     $ sudo apt-get update
   2.     $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   3.     $ apt-cache madison docker-ce
   4.     $ sudo docker run hello-world
<br>
   

## 使用步骤

1. 安装Docker
2. 去Docker仓库找到软件相应的镜像
3. 使用Docker运行镜像，这个镜像就会生成一个Docker容器
4. 对容器的启动停止就是对软件的启动停止

***

## 常用命令

### ***OPTIONS的命令***

* --name：为容器指定一个名字
* -d：以后台运行，即daemon运行，也叫守护式容器
* -i：以交互模式运行容器，通常与-t同时使用
* -t：为容器重新分配一个伪输入终端，通常与-i同时使用
* -P：大写P，随机端口映射
* -p：小写p，指定端口映射
  * ip：hostPort : containerPort
  * ip : :containerPort
  * hostPort : containePort
  * containerPort
  
### 重要命令

* ***docker rm -f $(docker  ps -a -q)***：一次删除多个容器
* ***docker run -d 容器名***：docker run -d 容器名
* ***docker logs -f -t --tail 容器ID***
  * -t：加入时间戳
  * -f：跟随最新的日志打印
  * --tail：数字显示最后多少条
* ***docker top 容器ID***：查看容器内运行的进程
* ***docker inspect 容器ID***：查看容器内部细节
* ***进入正在运行的容器并以命令行交互***
  * docker exec -it 容器ID bashShell
  * 重新进入docker attach 容器ID
  * 两者区别：attach直接进入容器命令启动命令的终端，不会启动新的进程,exec是在容器中打开新的终端，并且可以启动新的进程 
* ***docker cp 容器ID：容器内路径 目的主机路径***：将容器内的文件拷贝到主机上
* ***commit***
  * docker commit：提交容器副本 使之成为一个新的镜像
  * docker commit -m="提交的描述信息” -a="作者"  容器ID 要创建的，目标镜像名：[标签名] 

## ***Docker镜像***

### ***什么是镜像***

镜像是一种轻量级，可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所哟内容，包括代码，运行时，库，环境变量和配置文件。

### ***UnionFS（联合文件系统）***

是一种分层，轻量级并高性能的文件系统，支持对文件爱你系的修改作为一次提交来一层层的叠加，同时将不同目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础，镜像可以通过分层来进行继承，基于基础镜像，可以制造各种具体的应用镜像。

#### ***特性***

* 一次同时加载多个文件系统，但从外面看来，只能看到一个文件系统，连恶化加载会把各层文件系统叠加起来，这样最直接哦功能的文件系统会包含所有底层的文件和目录、
* docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为容器层，容器层之下都叫镜像层。

## ***容器***

### ***什么是容器****

* 容器镜像是轻量的、可执行的独立软件包 ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
* 容器化软件适用于基于 Linux 和 Windows 的应用，在任何环境中都能够始终如一地运行。
* 容器赋予了软件独立性　，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

### ***容器的特点***

* 轻量：在一台机器上运行的多个 Docker 容器可以共享这台机器的操作系统内核；它们能够迅速启动，只需占用很少的计算和内存资源。镜像是通过文件系统层进行构造的，并共享一些公共文件。这样就能尽量降低磁盘用量，并能更快地下载镜像
* 标准：Docker 容器基于开放式标准，能够在所有主流 Linux 版本、Microsoft Windows 以及包括 VM、裸机服务器和云在内的任何基础设施上运行。
* 安全：Docker 赋予应用的隔离性不仅限于彼此隔离，还独立于底层的基础设施。Docker 默认提供最强的隔离，因此应用出现问题，也只是单个容器的问题，而不会波及到整台机器。
  
### ***容器数据卷***

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能绕过Union File System提供的一些用于持久存储或共享数据的特性。
卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此docker不会在容器删除时删除其挂载的数据卷




















        

