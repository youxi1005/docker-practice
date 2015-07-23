#  [Docker 实践 2：用 Docker 搭建 hg-server](http://blog.csdn.net/lincyang/article/details/43450999)

如果有人已经将 hg server 的 image 做好了，那么我还要自己作吗？答案是拿来用吧。

## 一、安装

用 hg 为关键词搜索，得出以下结果：

```

    $ docker search hg
    NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    hgomez/gatling                                                                    1                    [OK]
    v7soft/hgdns                                                                      0                    [OK]
    hg8496/gridvis-service                                                            0                    [OK]
    hgomez/di-centos6-myjenkins-lts                                                   0                    [OK]
    jrandall/hgi-project                                                              0                    [OK]
    hgomez/di-centos6-myartifactory                                                   0                    [OK]
    hgomez/di-centos6-myjenkins                                                       0                    [OK]
    hgomez/di-centos6-mynexus                                                         0                    [OK]
    hgomez/di-centos6-myarchiva                                                       0                    [OK]
    hg8496/piwigo                                                                     0                    [OK]
    hg8496/apache                                                                     0                    [OK]
    hgomez/di-centos6-mygitblit                                                       0                    [OK]
    hgomez/di-centos6-mygitbucket                                                     0                    [OK]
    jyotisingh/ubuntu-hg                                                              0                    
    hg8496/dokuwiki                                                                   0                    [OK]
    hg8496/owncloud                                                                   0                    [OK]
    misshie/ucsc-blat-hg19                                                            0                    [OK]
    ussie/hg-exec                     adds mercurial to ubuntu:14.04.                 0                    
    misshie/ucsc-blat-hg38                                                            0                    [OK]
    hg8496/gridvis-pc                                                                 0                    [OK]
               Test. Automated builds for this repo are b...   0                    [OK]
    hg8496/rsync                                                                      0                    [OK]
    secondbit/hgbundler                                                               0                    
    uotbw/hgamer3d                    Docker image for hgamer3d, see www.hgamer3...   0                    
    hgomez/di-centos6-base                                                            0                    [OK]

```

hgweb 貌似不错的选择，在 github 上的主页是 https://github.com/amclain/docker-hgweb 。
将其 pull 下来，在漫长的等待中我也在思考着如何启动它。

主页上提供了它的 Dockerfile，通过它我们就可以了解这个 image 是如果构造的。先来说说什么是 Dockerfile。

## 二、Dockerfile

它是用户创建自定义镜像的文件。它通常分为四部分：基础镜像信息，维护者信息，镜像操作指令和容器启动时的指令。

```

    #基础系统信息，基于ubuntu 14.04构建的
    FROM ubuntu:14.04
    MAINTAINER Alex McLain <alex@alexmclain.com>
    RUN apt-get -qq update
    #安装apache、hg、php5
    RUN apt-get -y install apache2 apache2-utils curl mercurial php5 php5-cli php5-mcrypt
    # TODO: Remove
    #是的，vim确实很大，不安装为好
    RUN apt-get -y install vim
    RUN echo "colorscheme delek" > ~/.vimrc
    # Configure hgweb
    ADD hg/add.php /etc/default/hgweb/hg/
    ADD hg/hgweb.config /etc/default/hgweb/hg/
    ADD hg/hgweb.cgi /etc/default/hgweb/hg/
    ADD hg/hgusers /etc/default/hgweb/hg/
    # Configure Apache
    ADD apache/hg.conf /etc/default/hgweb/apache/
    RUN rm /etc/apache2/sites-enabled/*
    RUN a2enmod rewrite && a2enmod cgi
    ADD load-default-scripts /bin/
    RUN chmod u+x /bin/load-default-scripts
    #创建一个挂载点，本机或其他容器可以将其挂载。启动时用-v参数进行挂载
    VOLUME /var/hg
    VOLUME /etc/apache2/sites-available
    #暴露的端口号，启动时要通过-p参数指定
    EXPOSE 80
    #启动时执行的命令
    CMD load-default-scripts && service apache2 start && /bin/bash

```

## 三、启动

有了上述的背景，我们知道启动时要做两件事：指定端口号、挂载本地目录。

比如还是使用端口号 80，那么只需用 -p 80:80即可。

比如本机目录 hg-repos 用来做 hg repo 的放置目录，只需 -v /home/linc/hg-repos:/var/hg/repos 挂载即可。

另外，我们还要将其启动在后台（Daemonized），加上-d 参数。

完整启动命令如下：

```

    docker run -idt -p 80:80 -v /home/linc/hg-repos:/var/hg/repos amclain/hgweb

```

## 四、与后台容器交互

1.attach 方法

docker 自带 attach 命令，但此命令的不方便之处在于，多个窗口（同时 attach 此容器）会同步显示操作，并且当一个窗口 exit 时，所有窗口都会退出，后台运行的容器也停止了。

```

    $ docker ps
    CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                  NAMES
    b22cc1880b7a        amclain/hgweb:latest            "/bin/sh -c 'load-de   3 hours ago         Up 3 hours          0.0.0.0:80->80/tcp     high_almeida  

    $ docker attach b22cc1880b7a
    root@b22cc1880b7a:/# 

```

2.nsenter

此工具需要从源码安装：

```

    $ cd /tmp; curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
    $ ./configure --without-ncurses
    $ make nsenter && sudo cp nsenter /usr/local/bin

```

直接用 nsenter 命令交互很繁琐，然后有人写了配置文件放到 bashrc 中，就可以方便的使用了。

```

    #docker
    export DOCKER_HOST=tcp://localhost:4243
    alias docker-pid="sudo docker inspect --format '{{.State.Pid}}'"
    alias docker-ip="sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}'"
    
    #the implementation refs from https://github.com/jpetazzo/nsenter/blob/master/docker-enter
    function docker-enter() {
    if [ -e $(dirname "$0")/nsenter ]; then
    # with boot2docker, nsenter is not in the PATH but it is in the same folder
    NSENTER=$(dirname "$0")/nsenter
    else
    NSENTER=nsenter
    fi
    [ -z "$NSENTER" ] && echo "WARN Cannot find nsenter" && return
    
    if [ -z "$1" ]; then
    echo "Usage: `basename "$0"` CONTAINER [COMMAND [ARG]...]"
    echo ""
    echo "Enters the Docker CONTAINER and executes the specified COMMAND."
    echo "If COMMAND is not specified, runs an interactive shell in CONTAINER."
    else
    PID=$(sudo docker inspect --format "{{.State.Pid}}" "$1")
    if [ -z "$PID" ]; then
    echo "WARN Cannot find the given container"
    return
    fi
    shift
    
    OPTS="--target $PID --mount --uts --ipc --net --pid"
    
    if [ -z "$1" ]; then
    # No command given.
    # Use su to clear all host environment variables except for TERM,
    # initialize the environment variables HOME, SHELL, USER, LOGNAME, PATH,
    # and start a login shell.
    #sudo $NSENTER "$OPTS" su - root
    sudo $NSENTER --target $PID --mount --uts --ipc --net --pid su - root
    else
    # Use env to clear all host environment variables.
    sudo $NSENTER --target $PID --mount --uts --ipc --net --pid env -i $@
    fi
    fi
    }

```

其中有两个 alias 和一个 function，使用 docker-enter 会很容易于容器交互并没有 attach 中的副作用。如下：

```

    $ docker ps
    CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                  NAMES
    beb178cd9335        amclain/hgweb:latest            "/bin/sh -c 'load-de   11 seconds ago      Up 10 seconds       0.0.0.0:80->80/tcp     stoic_yonath          

    $ docker-enter beb178cd9335
    root@beb178cd9335:~# ls
    root@beb178cd9335:~# pwd
    /root

```

## 五、快速启动 hg-server

咱也写个 alias 放子 bashrc 中，如下：

```

    alias docker-load-hg-server="sudo docker run -idt -p 80:80 -v /home/linc/hg-repos:/var/hg/repos amclain/hgweb"

```

启动它：

```

    $ docker-load-hg-server 
    [sudo] password for linc: 
    beb178cd933502970fd12d9a4babecef5475a52d85a207066c665b4a620c5a62

```

**改进**

对于文件的挂载，其实直接挂镜像的/var/hg更好，这样里面的几个配置文件如 hgusers  hgweb.cgi  hgweb.config，我们可以直接进行配置。

```

    sudo docker run -idt -p 80:80 -v /home/linc/hg-repos:/var/hg amclain/hgweb

```

版权声明：本文为博主原创文章，未经博主允许不得转载。
    





