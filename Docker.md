# Docker

## 概念

Docker 能够自动执行重复性任务，例如搭建和配置开发环境，从而解放了开发人员以便他们专注在真正重要的事情上：构建杰出的软件。
用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

**docker主机(host)** : 安装了Docker程序的机器（docker直接安装在操作系统之上  
**docker客户端（Client）**：链接docker主机进行操作  
**docker仓库（Repository）**：用来保存各种打包好的软件镜像  
**docker镜像（Images）**：软件打包好的镜像，放在docker仓库中  
**docker容器（Container）**：镜像启动后的实例称为一个容器，容器是独立运行的一个或一组应用

## 安装Docker

官方教程  
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
  
### **重要命令**

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

***

## **Docker镜像**

### ***什么是镜像***

**操作系统分为内核和用户空间。**对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 root 文件系统。  
**Docker 镜像是一个特殊的文件系统**，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。 镜像不包含任何动态数据，其内容在构建之后也不会被改变。  
Docker 设计时，就充分利用 Union FS的技术，将其设计为 分层存储的架构 。 镜像实际是由多层文件系统联合组成。

### ***UnionFS（联合文件系统）***

是一种分层，轻量级并高性能的文件系统，支持对文件爱你系的修改作为一次提交来一层层的叠加，同时将不同目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础，镜像可以通过分层来进行继承，基于基础镜像，可以制造各种具体的应用镜像。

#### ***特性***

* 一次同时加载多个文件系统，但从外面看来，只能看到一个文件系统，连恶化加载会把各层文件系统叠加起来，这样最直接哦功能的文件系统会包含所有底层的文件和目录、
* docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为容器层，容器层之下都叫镜像层。

***

## **容器**


### ***什么是容器****

* 容器镜像是轻量的、可执行的独立软件包 ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
* 容器化软件适用于基于 Linux 和 Windows 的应用，在任何环境中都能够始终如一地运行。
* 容器赋予了软件独立性　，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。
* 容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。前面讲过镜像使用的是分层存储，容器也是如此。
* 容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

### ***容器的特点***

* 轻量：在一台机器上运行的多个 Docker 容器可以共享这台机器的操作系统内核；它们能够迅速启动，只需占用很少的计算和内存资源。镜像是通过文件系统层进行构造的，并共享一些公共文件。这样就能尽量降低磁盘用量，并能更快地下载镜像
* 标准：Docker 容器基于开放式标准，能够在所有主流 Linux 版本、Microsoft Windows 以及包括 VM、裸机服务器和云在内的任何基础设施上运行。
* 安全：Docker 赋予应用的隔离性不仅限于彼此隔离，还独立于底层的基础设施。Docker 默认提供最强的隔离，因此应用出现问题，也只是单个容器的问题，而不会波及到整台机器。
  
## 容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能绕过Union File System提供的一些用于持久存储或共享数据的特性。
卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此docker不会在容器删除时删除其挂载的数据卷  

### **特点**

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止  

### **功能**

1. 容器的持久化
2. 容器间继承+共享数据

### **使用**

* 直接命令添加
  * 添加命令：docker run -it -v/宿主机绝对路径目录:/容器内目录 镜像名
* DockerFile添加
  * 根目录下新建mydocker文件夹并进入
  * 可在DockerFile中使用VOLUME指令来给镜像添加一个或多个数据卷
  * File构建
  * build后生成镜像
  * run容器  

***

## ***Dockerfile***

### ***构建过程***

***Dockerfile基础知识***

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

### **流程**

从应用软件的角度来看，Dockerfile，Docker镜像与Docker容器分别代表软件的三个阶段  

**1. Dockerfile是软件的原材料**  
需要定义以恶搞DockerFile，DockerFile定义了进程所需的一切东西，包括执行代码或是文件，环境变量，依赖包，运行时环境，动态链接库，操作系统的发行版，服务进程和内核进程等等。

**2. Docker镜像是软件的交付品**  
在用Docker定义了一个文件之后，docker build时会产生一个镜像，当运行镜像时，会真正开始提供服务。  

**3. Docker容器可以认为是软件的运行态**  
直接提供服务

DockerFile面向开发，Docker镜像称为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。  

![docker体系](https://s2.ax1x.com/2019/11/19/McvPII.png)  

## **Dockerfile体系结构**

Docker中mysql镜像的Dockerfile文件
``` 
FROM debian:stretch-slim

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr && rm -rf /var/lib/apt/lists/*

# add gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& gpgconf --kill all \
	&& rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y --auto-remove ca-certificates wget

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-get update && apt-get install -y --no-install-recommends \
# for MYSQL_RANDOM_ROOT_PASSWORD
		pwgen \
# for mysql_ssl_rsa_setup
		openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
		perl \
	&& rm -rf /var/lib/apt/lists/*

RUN set -ex; \
# gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
	key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	gpg --batch --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME"; \
	apt-key list > /dev/null

ENV MYSQL_MAJOR 8.0
ENV MYSQL_VERSION 8.0.18-1debian9

RUN echo "deb http://repo.mysql.com/apt/debian/ stretch mysql-${MYSQL_MAJOR}" > /etc/apt/sources.list.d/mysql.list

# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
	} | debconf-set-selections \
	&& apt-get update && apt-get install -y mysql-community-client="${MYSQL_VERSION}" mysql-community-server-core="${MYSQL_VERSION}" && rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
	&& chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	&& chmod 777 /var/run/mysqld

VOLUME /var/lib/mysql
# Config files
COPY config/ /etc/mysql/
COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]
```

**Dockerfile中的保留字指令**

* **FROM**：基础镜像，当前新镜像是基于哪个镜像的
* **MAINTIANER**：镜像维护者
* **RUN**：容器构建时需要运行的命令
* **EXPOSE**：当前容器对外暴露出的端口
* **WORKDIR**：在创建容器后，终端默认登陆的目录
* **ENV**：用来构建镜像过程中设置环境变量
* **ADD**：将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包
* **COPY**：类似ADD，拷贝文件和目录到镜像中，将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。
* **VOLUME**：容器数据卷，用于数据保存和持久化工作。
* **CMD**：指定一个容器启动时要运行的命令，Dockerfile中可以有多个CMD命令，但只有最后一个会生效，CMD会被docker run之后的参数替换。
* **ENTRYPOINT**：与CMD类似，指定一个容器启动时要运行的命令，但是ENYRYPOINT可以追加指令而不会被覆盖。
* **ONBUILD**：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发。

