- [Docker 原理及安装](#docker-原理及安装)
  - [三大特征](#三大特征)
  - [架构](#架构)
  - [Docker 运行原理](#docker-运行原理)
  - [centos6 安装](#centos6-安装)
  - [centos7 安装](#centos7-安装)
  - [Docker 镜像](#docker-镜像)
- [Docker 命令](#docker-命令)
  - [帮助命令](#帮助命令)
  - [镜像命令](#镜像命令)
  - [容器命令](#容器命令)
- [Docker 容器数据卷](#docker-容器数据卷)
  - [作用](#作用)
  - [添加数据卷](#添加数据卷)
  - [数据卷容器](#数据卷容器)
- [DockerFile](#dockerfile)
  - [概念](#概念)
  - [Dockerfile 的解析过程](#dockerfile-的解析过程)
  - [Dockerfile 体系结构(保留字指令)](#dockerfile-体系结构保留字指令)
  - [实例](#实例)
    - [自定义centos](#自定义centos)
    - [验证 `CMD` 和 `ENTRYPOINT` 的区别](#验证-cmd-和-entrypoint-的区别)
    - [验证 `ONBUILD` 的作用](#验证-onbuild-的作用)
    - [自定义 `tomcat9`](#自定义-tomcat9)
- [DockerCompose](#dockercompose)
- [Docker 网络模式](#docker-网络模式)
  - [主机模式 hostonly](#主机模式-hostonly)
- [一些错误解决](#一些错误解决)
  - [容器内部无法访问网关](#容器内部无法访问网关)

<!-- 本文档由conradpeng整理，转载请注明出处 -->
<!-- https://github.com/conradpeng -->

# Docker 原理及安装

## 三大特征

> 镜像 容器 仓库

- ### 镜像 `Image`
    镜像是一个只读的模板<br>
    可以`用来创建 docker 容器`，一个镜像可以创建很多容器

- ### 容器 `Container`
    容器是 `镜像创建的运行实例`<br>
    `docker` 利用容器运行一个或一组应用

    每个容器都是一个集装箱，可以相互隔离，保证安全

    `可以把容器看作是简易版的 linux 环境`，(包含root用户权限，进程空间，用户空间和网络空间等）和运行在其中的应用程序

- ### 仓库 `Repository`
    仓库是 `集中存放镜像的地方`
    分为公开库(dockerhub)和私有库

    仓库注册服务器往往存放多个仓库，每个仓库包含多个镜像，每个镜像有不同的 `tag`

## 架构

- ### 架构图
    ![docker架构图](https://www.runoob.com/wp-content/uploads/2016/04/576507-docker1.png)

- ### 分析
    >`Client`：`docker`客户端，即对`docker`操作的命令终端<br>
    >`Hosts`：即运行`docker`所在的运行系统，主机<br>
    >`local host`: 本地主机<br>
    >`remote host`： 远程主机<br>
    >`Registries`: 注册仓库<br>
    >`Docker hub`: 公开库<br>private registry`: 私有仓库

    `Docker`本身是一个容器运行的载体，可以被称作管理引擎，即`client`<br>

    我们使用 `client`，从注册仓库上拉取我们需要的镜像到主机上，这些镜像的本质就是应用程序和配置依赖打好包形成一个可以交付的运行环境，<br>

    我们使用镜像去运行一个个实例(即容器)，这些容器像是一个个独立的集装箱，我们在主机上将这些实例整合起来，相互协作，构建一个运行环境


## Docker 运行原理

- ### 工作原理

    `Docker`是`CS`结构的系统<br>
    `Docker`守护进程运行在主机上，通过`Socket`连接从客户端访问，守护进程从客户端上接收命令并管理运行在主机上的容器

    客户端-守护进程-容器

- ### 与虚拟机的区别

    - `Docker`有更少的抽象层<br>
        虚拟机是软件+硬件构成的完整的操作系统<br>
        而`Docker`不需要实现硬件资源虚拟化，替换成了`Docker`引擎，没有硬件负担 (这也是镜像小的原因，镜像中不包含硬件资源，`docker`的`centos`镜像`200M`，虚拟机`centos`镜像`4G`)

    - `Docker`直接使用宿主机内核<br>
        新建容器时，<br>
        `Docker`不需要和虚拟机一样重新加载一个操作系统内核，直接使用宿主机的内核<br>
        新建虚拟机时，<br>
        虚拟机需要加载操作系统内核，耗时耗资源

    |           |    Docker容器     |   虚拟机    |
    |   ----    |         ----      |   ----     |
    | 操作系统   | 与宿主机共享OS     | 宿主机上运行虚拟机OS |
    | 存储大小   | 镜像小             | 镜像大 |
    | 性能       | 几乎无额外性能损失 | 操作系统额外的cpu、内存消耗 |
    | 移植性     | 轻便灵活、适合linux | 笨重 |
    | 硬件亲和性 | 面向软件开发者      | 面向硬件运维者 |
    | 部署       | 秒级               | 分级          |
    ||||



## centos6 安装

- ### 先决条件
    `docker` 只能运行在 `centos6` 以上的版本<br>
    `6.5` 以上要求系统 `64位`，内核版本 `2.6.32-431` 以上

    查看当前内核
    ``` sh
    $ uname -r
    ```

    查看已安装centos版本
    ``` sh
    $ cat /etc/redhat-release
    ```

- ### 安装依赖

     `Docker`使用 `EPEL`发布，先装 `EPEL`
    ``` sh
    # 查看可安装的包
    $ yum list | grep 'epel'
    epel-release.noarch                7-11                       extras

    # 安装 epel
    $ yum install -y epel-release.noarch
    ```

- ### 安装docker
    ```
    $ yum list | grep 'docker'

    $ yum install -y docker-io
    ```

    - 此时如果报错
        ```
        Cannot retrieve metalink for repository: epel/x86_64. Please verify its path and try again
        ```
        其原因是 `/etc/yum.repos.d/epel.repo` 里的镜像源路径不对 [解决方案](https://www.fujieace.com/linux/epel-x86_64.html)

        ``` sh
        # 1.在这个文件里添加一行 
        /etc/resolv.conf
        ↓        
        nameserver 114.114.114.114

        # 2.在这个文件里把 metalink 注释掉，baseurl 放开
        /etc/yum.repos.d/epel.repo

        [epel]
        name=Extra Packages for Enterprise Linux 7 - $basearch
        baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch
        #metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
        ```
        此时就可以正常下载安装了

- ### 运行

    ``` sh
    # 运行
    $ service docker start

    # 查看运行状态
    $ service docker status

    # 验证docker version
    $ docker version
    ```

- ### 配置文件

    ```
    /etc/sysconfig/docker
    ```


## centos7 安装

[官方文档](https://docs.docker.com/engine/install/centos/#prerequisites)
[菜鸟文档](https://www.runoob.com/docker/centos-docker-install.html)


- ### 卸载旧版

    ``` sh
    $ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
    ```

- ### 安装依赖

    ``` sh
    $ yum install -y yum-utils
    ```

- ### 设置源地址
    ``` sh
    # 官方源地址
    $ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

    # 阿里源地址
    $ yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```

- ### 可以选择启用或者不启用 `nightly or test repositories`
    ``` sh
    # 启用
    yum-config-manager --enable docker-ce-nightly
    yum-config-manager --enable docker-ce-test
    # 不启用
    yum-config-manager --disable docker-ce-nightly
    yum-config-manager --enable docker-ce-test
    ```

- ### 安装docker

    ``` sh
    # 直接安装最新版
    $ yum install docker-ce docker-ce-cli containerd.io

    # 或者选择版本安装
    $ yum list docker-ce --showduplicates | sort -r
    $ yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
    ```

- ### 运行

    ``` sh
    $ systemctl start docker

    # 查看状态
    $ systemctl status docker
    ```

- ### 配置文件
    ```
    /etc/docker/daemon.json
    ```

- ### 镜像加速

    登录阿里云开发者平台，搜索 `容器镜像服务`，找到`镜像加速器` [链接](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)


    根据文档的描述创建并修改配置文件
    
    ``` sh
    # centos7
    /etc/docker/daemon.json
    
    # centos6
    /etc/sysconfig/docker
    ```
    
    检查是否成功，查看`Registry Mirrors:` 是否修改成功
    ```
    $ docker info
    ```

## Docker 镜像
- ### 镜像是什么
    > 镜像是一种轻量级的、可执行的独立软件包，`用来打包软件运行环境和基于运行环境开发的软件`，它包含运行某个软件所需的所有内容，包括代码、运行库、环境变量、配置文件等等

- ### 联合文件系统 ( UnionFS )
    ![联合文件系统](https://upload-images.jianshu.io/upload_images/2441456-74c323a4b53e341f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    >分层、轻量级、高性能的文件系统<br>
    `支持对文件系统的修改作为一次提交来一层层叠加`，同时将不同的目录挂载到同一虚拟文件系统下<br>
    是`Docker`镜像的基础，镜像通过分层进行继承，基于基础镜像，制作各种具体的应用镜像

    特性：<br>
    一次同时加载多个文件系统，<br>
    从外面看来只能看到一个文件系统，但其 `本质是把各层文件系统叠加起来`，最终文件系统会包含所有底层的文件和目录


- ### Docker 镜像加载原理
    `Docker` 的镜像由一层层的文件系统组成，就是 `UnionFS`<br>

    > 主要是两部分: <br>

    > `bootfs`( `boot file system` boot文件系统)<br>
    `Docker 镜像的最底层就是 bootfs`<br>
    主要包含 `bootloader`( 加载器 )和 `kernel`( 内核 )<br>
    同linux系统一样，linux启动时，会加载`bootfs`文件系统，`bootloader` 的作用是引导加载 `kernel`，在`boot`加载完成后，整个内核都在内存之中了，内存的使用权由`bootfs`转移给了内核，系统卸载`bootfs`

    > `rootfs` ( `root file system` root文件系统 )<br>
    在`bootfs`之上，包含目录与文件( `/dev,/proc,/bin,/etc` 等等 )<br>
    即不同的操作系统发行版，如`ubuntu`， `centos`等


    > `docker`的镜像之所以非常小，是因为
    > - `rootfs`可以非常小，它只需要包含基本的命令、工具和程序库，无需硬件
    > - 底层直接使用宿主机的内核，对于不同的`linux`发行版，`bootfs`基本一致，`bootfs`共用即可

- ### 镜像分层
    每一个镜像都是层层叠加的结构，以`tomcat`为例<br>
    它的结构可能如下，最外层是我们直观看到的`tomcat`，其内部却包含了它的运行环境<br>
    tomcat<br>
    jdk7<br>
    centos<br>
    kernel

- ### 构建分层结构的原因
    > 最大的好处就是可以进行资源共享，镜像中的每一层都可以成为构成其他镜像的一部分<br>

    比如：多个镜像依赖于相同的`base`镜像，那么宿主机硬盘上只需要有一份`base`镜像就可以了，内存中也只需要加载一份`base`镜像就可以为所有容器进行服务，镜像的每一层都具有这个特点


# Docker 命令

## 帮助命令

- ### docker info

- ### docker version

- ### docker --help

## 镜像命令

- ### 查看本地镜像列表 `docker images`

    - 语法
        ```
        docker images [options][image]
        ```
        > `REPOSITORY` : 镜像的仓库源<br>
        `TAG` : 镜像的标签<br>
        `IMAGE ID` : 镜像的id<br>
        `CREATED` : 镜像的创建时间<br>
        `SIZE` : 镜像的大小

        使用 `REPOSITORY:TAG` 来定义不同的镜像

    - 参数<br>
        > `-a` : `all` 列出本地含中间镜像层<br>
        > `-q` : `quiet` 只显示镜像ID，常用于批量操作命令<br>
        > `--no-trunc` : 显示镜像的完整信息 ( 完全长度的 `IMAGE ID` )<br>
        > `-digests` : 显示镜像的摘要信息

- ### 搜索DockerHub上的镜像 `docker search`

    - 语法
        ```
        docker search [options] 
        ```
    - 参数
        > `--no-trunc` 显示镜像完整的描述信息
        ``` docker
        docker search --no-trunc apache
        ```

        > `-s` : `star` 过滤出点赞数超过某个数字的镜像列表
        ``` docker
        docker search -s 30 apache
        ```

        > `--automated` : 罗列出 `automated build` 类型的镜像
        ``` docker
        docker search --automated apache
        ```

- ### 拉取DockerHub上的镜像 `docker pull`

    - 语法<br>
        ```
        docker pull 镜像名[:TAG]
        ```
        如果没有后跟版本号，则默认拉取最新的版本
        ``` docker
        # 下面两句命令等价
        docker pull tomcat
        docker pull tomcat:latest
        ```
- ### 删除本地镜像 `docker rmi`

    - 语法<br>
        ```
        docker rmi 镜像名[:TAG] 
        ```
        `rmi` 即 `remove images`<br>
        如果没有后跟版本号，则默认拉取最新的版本
        ``` docker
        # 以下两句命令等价
        docker rmi hello-world
        docker rmi hello-world:latest
        ```
        如果有容器正在使用，则需要先删除容器再删除镜像
        ``` docker
        # 查看容器列表
        docker ps -a
        # 删除对应的容器
        docker rm 2ab123b75484
        # 删除镜像
        docker rmi hello-world
        # 或者强制删除
        docker rmi -f hello-world
        ```

    - 参数<br>
      
        >`-f` : `force` 强制删除<br>
    
    - 其他用法<br>
        > 同时删除多条
        ``` docker
        docker rmi 镜像1[:TAG] 镜像2[:TAG]
        docker rmi -f hello-world nginx
        ```
        > 检索条件删除
        ``` docker
        docker rmi $(条件)
        # 例:删除本地全部镜像
        docker rmi -f $(docker image -qa)
        ```

- ### 提交容器副本使其成为新的镜像 `docker commit`
    - 语法
        ```
        docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像[:标签名]
        ```
        ``` docker
        # 生成的镜像名可以使用命名空间
        docker commit -m="默认页面" -a="jack" 3ae70c8727f6 jack/tomcat:0.1
        ```

    - 参数
        > `-m="描述信息"`<br>
        > `-a="作者"`

- ### 镜像提交到云端
    - 容器生成新的本地镜像
        ``` docker
        docker commit -a="conrad" -m="centos with tomcat and java" 5514e763e80c conrad/mycentos:1.0
        ```
    - 阿里云镜像仓库<br>
        直接登录阿里云创建镜像仓库<br>
        创建完成选择管理会有详细的操作步骤

        ``` docker
        # 登录阿里云
        docker login --username=用户名 registry.cn-hangzhou.aliyuncs.com

        # 从仓库拉取镜像
        docker pull registry.cn-hangzhou.aliyuncs.com/xxxx/mycentos:[镜像版本号]

        # 标记本地镜像，将其归入云端仓库
        docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xxxx/mycentos:[镜像版本号]

        # 推送镜像至仓库
        docker push registry.cn-hangzhou.aliyuncs.com/xxxx/mycentos:[镜像版本号]
        ```

## 容器命令

- ### 新建并启动容器 `docker run`

    - 语法
        ```
        docker run [options] image [command][arg]
        ```
    - 参数
        > `--name="容器新名字"` : 为容器指定一个名字，不加该参数时会默认给容器一个名字<br>
        ``` docker
        docker run -it --name="mycentos" centos
        ```
        >`-d` : 后台运行容器，并返回容器 `ID`，守护式启动<br>
        ```
        [root@localhost ~]# docker run -d centos
        78afff9f3061518d5896a9de70ff30310a905511deeb65025fd1b9a7cd1fb6f7
        [root@localhost ~]# docker ps -a
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
        78afff9f3061        centos              "/bin/bash"         8 seconds ago       Exited (0) 7 seconds ago                            angry_gould
        ```
        > `-i` : 交互模式运行<br>
        会提供一个命令行与容器交互
        ```
        [root@localhost ~]# docker run -i centos
        ps -ef
        UID        PID  PPID  C STIME TTY          TIME CMD
        root         1     0  0 03:08 ?        00:00:00 /bin/bash
        root         6     1  0 03:08 ?        00:00:00 ps -ef
        ```
        >`-t` : 为容器分配一个伪输入终端<br>
        但是不具备交互功能
        ```
        [root@localhost ~]# docker run -t centos
        [root@a3325920717e /]# ps -ef
        ^C^C
        ```
        > `-it` : `it` 参数常结合在一起使用，启动交互式容器<br>
        ```
        [root@localhost ~]# docker run -it centos
        [root@a3874b2d0c59 /]# ps -ef
        UID        PID  PPID  C STIME TTY          TIME CMD
        root         1     0  0 03:14 pts/0    00:00:00 /bin/bash
        root        14     1  0 03:14 pts/0    00:00:00 ps -ef
        ```
        > `-P` : 随机端口映射<br>
        >`-p` : 指定端口映射，主机端口:容器端口<br>
        ```
        docker run -it -p 8888:8080 tomcat
        ```
        > `-v` : 添加容器卷<br>

        > `command` : 指交互的命令模式
        ``` docker
        # 不携带的情况下默认指 /bin/bash
        docker run -it centos /bin/bash
        ```

- ### 列出容器列表 `docker ps`
    - 语法<br>
        ```
        docker ps [options] 
        ```
        不携带参数默认会只显示出正在运行的容器
        ``` docker
        docker ps
        ```
        > `CONTAINER ID` : 容器 `ID`<br>
        > `IMAGE` : 镜像名<br>
        > `COMMAND` : 命令模式<br>
        > `CREATED` : 创建时间<br>
        > `STATUS` : 状态 `up` 运行中 `Exited`停止中<br>
        > `PORTS` : 端口<br>
        > `NAMES` : 容器名<br>

    - 参数
        > `-l` : `last` 查看上一个容器<br>
        > `-n` : `number` 查看上 `n` 个容器列表<br>
        ``` docker
        # 取得最近运行的5个容器的列表
        docker ps -n 5
        ```
        > `-a` : `all` 查看所有的容器列表<br>
        > `-q` : `quiet` 查看容器 `ID`，常用于批量操作<br>
        > `--no-trunc` : 显示镜像的完整信息

- ### 退出容器 `ctrl+P+Q`
    - `exit` 容器停止退出
    - `ctrl+P+Q` 容器不停止退出

- ### 启动容器 `docker start`
    - 语法
        ```
        docker start 容器ID
        ```
    - 不携带参数的情况下，`start` 命令会是后台运行容器
    - 参数
      
        > `-i` : 启动交互式容器
- ### 重启容器 `docker restart`
- ### 停止容器 `docker stop`
- ### 强制停止 `docker kill`
- ### 查看容器内部进程 `docker top`
- ### 查看容器内部细节 `docker inspect`
    - 返回一个 `json` 串，包含容器的各种详细配置信息
- ### 删除容器 `dcoker rm`
    - 语法
        ```
        dcoker rm 容器ID1 [容器ID2]
        ```
    - 参数

        > `-f` : `force` 强制删除

    - 其他用法

        ``` docker
        # 删除所有容器
        docker rm -f $(docker ps -qa)
        docker ps -qa | xargs docker rm -f
        ```
- ### 查看容器日志 `docker logs`

    - 语法
        ```
        docker logs -f -t --tail 容器ID
        ```
    - 参数
        > `-f` : `forward` 持续刷新输出<br>
       ```
        [root@6c57b298d26a ~]# while true; do echo 111; sleep 3; done;

        [root@localhost ~]# docker logs -f 6c57b298d26a
        [root@6c57b298d26a ~]# while true; do echo 111; sleep 3; done;
        111
        111
        111
        111
        111
        111
        111
        ...
        ...
       ```
        > `-t` : `time` 输出时间信息<br>
        ```
        [root@localhost ~]# docker logs -t 6c57b298d26a
        2020-11-04T01:41:25.709110632Z [root@6c57b298d26a ~]# while true; do echo 111; sleep 3; done;
        2020-11-04T01:41:25.709125414Z 111
        2020-11-04T01:41:28.709979528Z 111
        2020-11-04T01:41:31.711747607Z 111
        2020-11-04T01:41:34.713918928Z 111
        2020-11-04T01:41:37.714750331Z 111
        ```
        > `--tail` : 输出末尾的几行
        ```
        [root@localhost ~]# docker logs -tf --tail 5 6c57b298d26a
        2020-11-04T01:45:34.828906039Z 111
        2020-11-04T01:45:37.830723510Z 111
        2020-11-04T01:45:40.832663279Z 111
        2020-11-04T01:45:43.834852182Z 111
        2020-11-04T01:45:46.835911935Z 111
        ```
        > 结合使用，能够持续输出带有时间信息的log



- ### 重新以交互式进入容器 `docker attach/exec`
    - 语法<br>
        `attach`
        ```
        docker attach 6c57b298d26a
        ```
        `exec`
        ```
        docker exec -it 6c57b298d26a /bin/bash
        ```

- ### 在宿主机中对容器内部进行操作 `docker exec`
    - 语法
        ```
        docker exec [options] 容器ID
        ```
    - 参数
        > `d` : 后台运行<br>
        > `i` : 交互模式<br>
        > `t` : 伪终端

        ``` docker
        # 可以在宿主机中操作容器内部
        $ docker exec -d 6c57b298d26a ls /tmp
        $ docker exec -i 6c57b298d26a ls /tmp
        ks-script-2n9owwnh
        ks-script-xm1o5azb
        $ docker exec -t 6c57b298d26a ls /tmp
        ks-script-2n9owwnh  ks-script-xm1o5azb
        ```
        ``` docker
        # 例：可以以交互模式执行容器内的脚本
        docker exec -it mynginx /bin/sh /root/runoob.sh
        ```

- ### 容器内数据拷贝至主机 `docker cp`
    - 语法
        ```
        docker cp 容器ID:容器文件路径 主机目标路径
        ```
        使用 `cp` 命令拷贝出来的文件，文件信息与原文件完全一致
        ```
        docker cp 6c57b298d26a:/tmp/aa.log /tmp/aa2.log
        ```

- ### 更新容器配置 `docker update`
    - 语法
        ```
        docker update [options] 容器ID [容器ID...]
        ```
    - 参数
        > `--restart` : 当容器退出时重新启动的策略<br>
        `--memory-swap` : 交换空间，“-1”允许无限交换<br>
        `--memory , -m` : 内存限制

        配置容器自启、重启策略
        ```
        docker update --restart=always 6c57b298d26a
        ```

# Docker 容器数据卷

## 作用
- 容器持久化
- 容器继承+共享数据 (可以理解为虚拟机的映射)

## 添加数据卷

- ### 运行容器并添加 `docker run -v`
    - 语法<br>
        会自动在宿主机和容器内创建目录 默认情况下是可读可写<br>
        
        ``` docker
        docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名
        # 例
        docker run -d --name="tom_volume" --net=host -v /myDataVolume:/containerDataVolume pk/tomcat:0.1
        ```
        可以通过`docker inspect` 验证容器配置中是否添加了容器卷
        
    - 权限<br>
        > `:ro` : `read only` 只读 (主写从读)
        ``` docker
        docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名
        ```
    
- 容器重启时会重新加载容器卷，数据依旧同步
  
    - 参数
      
        > `--privileged==true` : 挂载主机目录，报无权限错误时，可携带在主机目录后再运
    
- ### 使用 DockerFile 添加 `VOLUNE`

    - 语法<br>
        `Dockerfile` 中添加
        ``` dockerfile
        VOLUME ["/container1Data", "/container2Data"]
        ```
        为了不影响可移植性，`dockerfile` 设置容器卷不能设置主机目录，在运行时会自动分配一个主机目录
        ``` yaml
        #使用docker inspect查看配置
        "Mounts": [
            {
                "Type": "volume",
                "Name": "188fca27f0506e0da608d505ea571b771a788a6b11a7eb1889a3550d3f728d95",
                "Source": "/var/lib/docker/volumes/188fca27f0506e0da608d505ea571b771a788a6b11a7eb1889a3550d3f728d95/_data",
                "Destination": "/container1Data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
        ]
        ```

## 数据卷容器 
- ### 概念
    >命名的容器挂载数据卷，其他的容器通过挂载该容器实现数据共享<br>
    >挂载数据卷的容器，被称为 `数据卷容器`<br>

    >父容器和所有子容器，包括子容器之间都是数据共享的<br>
    即使父容器被删除，子容器之间依旧可以数据共享，因为 `数据卷的生命周期一直持续到没有容器使用它为止`

- ### 语法
    ``` docker
    docker run -it --volumes-from 父容器ID 镜像名
    ```
    注意前提是父容器已经挂载了数据卷
    ``` docker
    docker run -it --name="son1" --volumes-from 97595baec7a1 centos
    ```

# DockerFile

## 概念

- >`Dockerfile` 是用来构建 `Docker镜像` 的构成文件，是由一系列参数和命令构成的脚本

- > 运行三步骤<br>
    Dockerfile<br>
    > docker build<br>
    ``` docker
    docker build -f Dockerfile的路径 -t 新镜像名[:TAG] .
    ```
    > docker run

## Dockerfile 的解析过程

- ### 基础知识

    - 每条保留字指令必为大写字母，同时后面必须跟随至少一个参数
    - 指令由上至下顺序执行
    - `#` 用来注释
    - 每条指令都会创建一个新的镜像层，并对镜像进行提交

- ### 运行的大致流程

    - `docker` 从基础镜像 (`scratch` 根镜像) 运行一个容器
    - 执行一条指令并对容器做出修改
    - 执行类似于 `docker commit` 的操作提交一个新的镜像层
    - `docker` 再基于刚提交的镜像运行一个新的容器
    - 逐行执行 `Dockerfile` 中的指令直至完成

## Dockerfile 体系结构(保留字指令)

- ### 保留字指令一览
    | BUILD      | BOTH     | RUN        |
    | ----       | ----     | ----       |
    | FROM       | WORKDIR  | CMD        |
    | MAINTAINER | USER     | ENV        |
    | COPY       |          | EXPOSE     |
    | ADD        |          | VOLUME     |
    | RUN        |          | ENTRYPOINT |
    | ONBUILD    |          |            |
    |            |          |            |

- ### `FROM`
    基础镜像，当前的新镜像是基于哪个镜像的
    ``` docker file
    FROM centos
    ```
    
- ### `MAINTAINER`
    镜像维护者的名字和邮箱
    ``` dockerfile
    MAINTAINER conrad<conrad@xxxx.com>
    ```
    
- ### `RUN`
    容器构建时需要运行的命令
    ``` dockerfile
    RUN yum install -y net-tools
    ```
    
- ### `EXPOSE`
    当前容器对外暴露的端口
    ``` dockerfile
    EXPOSE 80
    ```
    
- ### `WORKDIR`
    登录容器时的默认工作目录
    ``` dockerfile
    WORKDIR /tmp
    ```
    
- ### `ENV`
    在构建镜像的过程中设置环境变量
    ``` dockerfile
    ENV MYPATH /usr/local
    WORKDIR $MYPATH
    ```
    
- ### `ADD`
    将宿主机目录下的文件拷贝进镜像并自动处理URL和解压
    
- ### `COPY`
    和 `ADD` 类似，拷贝文件到镜像中
    ``` dockerfile
    COPY src dest
    COPY ["src", "dest"]
    ```
    
- ### `VOLUME`
    容器数据卷
    ``` dockerfile
    VOLUME ["/container1Data", "/container2Data"]
    ```
    
- ### `CMD`
    指定容器启动时要运行的命令<br>
    `Dockerfile` 中可以设置多个 `CMD` 指令，但是只有最后一个会生效(后面会覆盖前面)，同时也会被 `docker run` 之后的 `command` 所覆盖
    ``` dockerfile
    CMD ["/bin/bash"]
    CMD ["aaa.sh", "run"]
    CMD ["curl", "-s", "https://www.baidu.com"]
    ```
    
- ### `ENTRYPOINT`
    指定容器启动时要运行的命令<br>
    区别于 `CMD` 的会被覆盖，`ENTRYPOINT` 则是追加指令
    ``` dockerfile
    # 用法同CMD
    ENTRYPOINT ["/bin/bash"]
    ```

- ### `ONBUILD`

    父镜像被子镜像继承时会触发父镜像的`ONBUILD`，常用于做一些收尾工作，后面可以跟其他保留字
    ``` dockerfile
    ONBUILD RUN echo "Triggered!!"
    ```



## 实例

### 自定义centos

- 需求<br>
    给精简版的 `centos` 安装 `yum` 和 `net-tools`

- Dockerfile<br>
    ``` dockerfile
    FROM centos
    WORKDIR /tmp/
    RUN yum install vim
    RUN yum install net-tools
    EXPOSE 80
    CMD /bin/bash
    ```

- 生成镜像
    ``` docker
    docker build -f /dockerfiles/test01_mycentos/Dockerfile -t conrad/centos:1.0 .
    ```

### 验证 `CMD` 和 `ENTRYPOINT` 的区别

- 思路<br>
    构建两个镜像，使用 `ls` 来获取文件信息<br>
    一个使用 `CMD` 执行，<br>
    一个使用 `ENTRYPOINT` 执行，<br>
    再分别验证 `docker run` 时再最后加 `-la参数` 是否会返回详细信息

- Dockerfile
    ``` dockerfile
    FROM centos
    CMD ["ls"]
    ```
    ``` dockerfile
    FROM centos
    ENTRYPOINT ["ls"]
    ```

- 生成镜像
    ``` sh
    # CMD
    docker -f /dockerfiles/test02_verify_diff/Dockerfile1 -t getdir1 .

    # ENTRYPOINT
    docker -f /dockerfiles/test02_verify_diff/Dockerfile2 -t getdir2 .
    ```

- 运行容器
    ``` sh
    # CMD
    docker run getdir1

    # ENTRYPOINT
    docker run getdir2
    ```
    此时运行结果一致，都是输出目录信息
    ``` sh
    bin
    etc
    games
    include
    lib
    ...
    ```

    ``` sh
    # CMD
    docker run getdir1 -l

    # ENTRYPOINT
    docker run getdir2 -l
    ```
    此时 `ENTRYPOINT` 可以正常返回头信息
    `CMD` 报错: `-l` 不是一个正常指令
    ``` sh
    [root@localhost test02_verify_diff]# docker run getdir1:cmd -l
    docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.
    ERRO[0001] error waiting for container: context canceled

    [root@localhost test02_verify_diff]# docker run getdir2:entrypoint -l
    total 0
    drwxr-xr-x 2 root root  6 May 11  2019 bin
    drwxr-xr-x 2 root root  6 May 11  2019 etc
    drwxr-xr-x 2 root root  6 May 11  2019 games
    drwxr-xr-x 2 root root  6 May 11  2019 include
    drwxr-xr-x 2 root root  6 May 11  2019 lib
    ...
    ```
    可以看到四次运行的容器各自的指令
    ``` sh
    # docker ps -a
    CONTAINER ID        IMAGE                COMMAND
    e47da539b656        getdir2:entrypoint   "ls -l"
    45788a723bc9        getdir1:cmd           "-l"
    9212a19307bf        getdir2:entrypoint   "ls"
    6dae320134e1        getdir1:cmd           "ls"
    ```

- 得出结论
    `CMD` 的指令会被 `run` 后面跟的指令覆盖<br>
    而 `ENTRYPOINT` 则是追加

### 验证 `ONBUILD` 的作用

- 父镜像 Dockerfile
    ``` dockerfile
    # dockerfile1
    FROM centos
    CMD ["/bin/bash"]
    ONBUILD RUN echo "Triggered!!"
    ```

- 生成镜像，新建继承
    ``` docker
    docker build -f /dockerfiles/test03_onbuild/Dockerfile1 -t onbuild .
    ```
    ``` dockerfile
    # dockerfile2
    FROM onbuild
    ```

- 运行子容器<br>
    可以看到在子容器运行时，触发了父镜像里设定的命令，打印出了对应信息
    ``` docker
    [root@localhost test03_onbuild]# docker build -f /dockerfiles/test03_onbuild/Dockerfile2 -t son .

    Sending build context to Docker daemon  3.072kB
    Step 1/1 : FROM onbuild
    # Executing 1 build trigger
    ---> Running in 5a1d0ef1e33a
    Triggered!!
    Removing intermediate container 5a1d0ef1e33a
    ---> f502787a022d
    Successfully built f502787a022d
    Successfully tagged son:latest
    ```

### 自定义 `tomcat9`

- Dockerfile
    ``` dockerfile
    FROM centos
    MAINTAINER conrad<conrad@xxx.com>
    # 解压并移动jdk和tomcat9到容器中
    ADD jdk-8u161-linux-x64.tar.gz /usr/local
    ADD apache-tomcat-9.0.39.tar.gz /usr/local
    RUN yum install -y vim
    ENV MYPATH /usr/local
    WORKDIR $MYPATH

    # 环境变量配置
    ENV JAVA_HOME /usr/local/jdk1.8.0_161
    ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.39
    ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.39
    ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

    # 端口
    EXPOSE 8080

    # 运行tomcat 同时监听log
    CMD /usr/local/apache-tomcat-9.0.39/bin/startup.sh && tail -f /usr/local/apache-tomcat-9.0.39/logs/catalina.out
    ```

- 运行并添加容器卷<br>
    添加的容器卷指向项目目录
    ``` docker
    docker run -d --name="mytom" -p 8888:8080 -v /vagrant_data/dockerfiles/test04_mytomcat9/myapp:/usr/local/apache-tomcat-9.0.39/webapps/myapp e8252f6d1dd2
    ```

- 验证<br>
    验证 `java`
    ``` sh
    [root@localhost test04_mytomcat9]# docker exec -it 5514e763e80c java -version
    java version "1.8.0_161"
    Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
    Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
    ```

    验证 `tomcat`，直接网页访问


# DockerCompose

# Docker 网络模式

## 主机模式 hostonly
```
docker run -d --net=host tomcat 
```

# 一些错误解决

## 容器内部无法访问网关

- ### 环境：<br>
    容器1 `ip 172.17.0.2` 端口映射 `8080:8080`<br>
    容器2 `ip 172.17.0.3` 端口映射 `8081:8080`<br>
    宿主机 网关 `172.17.0.1`

- ### 现象：<br>
    具体描述为<br>
    容器1和容器2之间可以互相ping通<br>
    但各自都无法ping通宿主机

- ### 分析：<br>
    `docker` 加载内核的`bridge.ko` 驱动异常，导致`docker0` 网卡无法转发数据包，也就是系统内核的网桥模块`bridge.ko` 加载失败导致的<br>
    [参考资料](https://blog.csdn.net/weixin_42288415/article/details/105366176)

- ### 解决：<br>
    [升级内核](https://www.cnblogs.com/xzkzzz/p/9627658.html)

    我这里装的是`kernel`
    ``` sh
    yum --enablerepo=elrepo-kernel install kernel
    ```

    同时也升级了 `kernel-devel` 和 `kernel-headers` ( 因为vbox的共享目录依赖内核 )
    ``` sh
    yum install "kernel-devel-uname-r == $(uname -r)"
    yum install -y kernel-headers-3.10.0-1127.19.1.el7.x86_644
    ```
    其他的步骤都一致