#  [Docker 实践](http://blog.csdn.net/lincyang/article/details/43055061)  

## 一、Docker 是什么

docker 直译为码头工人。当它成为一种技术时，做的也是码头工人的事。官网是这样描述它的：“Docker是一个开发的平台，用来为开发者和系统管理员构建、发布和运行分布式应用。”也就是说，如果把你的应用比喻为货物，那么码头工人（Docker）就会迅速的用集装箱将它们装上船。快速、简单而有效率。`
    
它是用 Go 语言写的，是程序运行的“容器”（Linux containers），实现了应用级别的隔离（沙箱）。多个容器运行时互补影响，安全而稳定。

我喜欢它的原因就是快速部署，安全运行，不污染我的系统。

## 二、试用 Try it！

官方提供一个互动的小教程，让你很容易的了解 Docker 的基本用法，快去[试试](https://www.docker.com/tryit/)吧！

## 三、安装

官方直接支持 64 位 Linux 系统安装 Docker，但如果想在 32 位系统中运行，有人也进行了一些尝试，比如 32Ubuntu 下，参考[点击打开链接](https://github.com/docker-32bit/ubuntu)。

其他系统的安装请参考[官网](https://docs.docker.com/installation/#installation)，下面说说我在 Ubuntu14.04 下的安装。     
 
1.将镜像加入到程序源中：

```

    ~$ sudo sh -c "echo deb http://mirror.yandex.ru/mirrors/docker/ docker main > /etc/apt/sources.list.d/docker.list"

```

2.接着 update

```

    $ sudo apt-get update

```

3.如果报错就 fix 掉它：

```

    W: GPG error: http://mirror.yandex.ru docker Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY D8576A8BA88D21E9\
```

解决此错误：

```

    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D8576A8BA88D21E9
    Executing: gpg --ignore-time-conflict --no-options --no-default-keyring --secret-keyring /tmp/tmp.RmJ1SUpsXX --trustdb-name /etc/apt/trustdb.gpg --keyring /etc/apt/trusted.gpg --primary-keyring /etc/apt/trusted.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D8576A8BA88D21E9
    gpg: requesting key A88D21E9 from hkp server keyserver.ubuntu.com
    gpg: key A88D21E9: public key "Docker Release Tool (releasedocker) <docker@dotcloud.com>" imported
    gpg: Total number processed: 1
    gpg:               imported: 1  (RSA: 1)

```

4.下载 docker：

```

    $ sudo apt-get install lxc-docker

```

静静的等待它下载完成吧。

另外，这个命令也许会有帮助：

```

    $ curl -sSL https://get.docker.com/ubuntu/ | sudo sh

```

## 四、初步使用

终端中输入 docker，打印出 docker 的命令列表：

```

    Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from a container's filesystem to the host path
    create    Create a new container
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    exec      Run a command in a running container
    export    Stream the contents of a container as a tar archive
    history   Show the history of an image
    images    List images
    import    Create a new filesystem image from the contents of a tarball
    info      Display system-wide information
    inspect   Return low-level information on a container
    kill      Kill a running container
    load      Load an image from a tar archive
    login     Register or log in to a Docker registry server
    logout    Log out from a Docker registry server
    logs      Fetch the logs of a container
    port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
    pause     Pause all processes within a container
    ps        List containers
    pull      Pull an image or a repository from a Docker registry server
    push      Push an image or a repository to a Docker registry server
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image to a tar archive
    search    Search for an image on the Docker Hub
    start     Start a stopped container
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Lookup the running processes of a container
    unpause   Unpause a paused container
    version   Show the Docker version information
    wait      Block until a container stops, then print its exit code

```

接下来就可以尝试使用这些命令了，不过在进行下一步之前，我们要先了解几个概念。

## 五、重要概念

1. image 镜像
镜像就是一个只读的模板。比如，一个镜像可以包含一个完整的 Ubuntu 系统，并且安装了 apache。
镜像可以用来创建 Docker 容器。
其他人制作好镜像，我们可以拿过来轻松的使用。这就是吸引我的特性。

2. Docker container 容器
Docker 用容器来运行应用。容器是从镜像创建出来的实例（好有面向对象的感觉，类和对象），它可以被启动、开始、停止和删除。

3. 仓库
这个好理解了，就是放镜像的文件的场所。比如最大的公开仓库是 Docker Hub。

## 六、几个简单的实践

1.search

搜索仓库中是否有 wordpress 这个博客镜像，如下：

```

    $ docker search wordpress
    NAME   DESCRIPTION STARS OFFICIAL   AUTOMATED
    wordpress  The WordPress rich content management syst...   185   [OK] 

```

2.下载这个镜像

```

    $ docker pull wordpress
    wordpress:latest: The image you are pulling has been verified

```

3.查看自己的镜像

```

    $ docker images
    REPOSITORY               TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    linc-wiki                latest              b5a1e34b01c2        27 hours ago        689.7 MB
    ubuntu                   latest              5ba9dab47459        5 days ago          188.3 MB
    wordpress                latest              ecc04d6d638c        6 days ago          470 MB

```

4.简单的运行

运行 wordpress 要进行 mysql 的配置，为了演示 run，将 ubuntu 跑起来吧。

```

    $ docker run -it ubuntu /bin/bash
    root@46ff2a695ce1:/# echo "I am linc"
    I am linc

```

至此，体验结束。后续会有更加精彩的实践等着我们，Docker，我们来了！

参考：  
https://docs.docker.com/installation/ubuntulinux/  
https://bitnami.com/stack/mediawiki  
https://docs.docker.com/userguide/  
https://dockerpool.com/static/books/docker_practice  
http://zhumeng8337797.blog.163.com/blog/static/1007689142014524115743806/  
http://www.cnblogs.com/imoing/p/dockervolumes.html  
https://github.com/docker/fig/issues/88  

版权声明：本文为博主原创文章，未经博主允许不得转载。

    


 