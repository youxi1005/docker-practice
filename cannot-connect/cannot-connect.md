# [Docker 实践 6：Cannot connect to the Docker daemon.](http://blog.csdn.net/lincyang/article/details/44156895)

正在免费适用着 Aliyun 主机，当然要用 docker 来部署我的服务器啦。但是今天碰到了题目的问题，细节如下：

```

    # docker info
    FATA[0000] Cannot connect to the Docker daemon. Is 'docker -d' running on this host? 
    # docker -d
    INFO[0000] +job serveapi(unix:///var/run/docker.sock)   
    FATA[0000] pid file found, ensure docker is not running or delete /var/run/docker.pid 

```

首先要查看 docker daemon 是否在运行。

```

    # ps aux | grep docker
    root   691  0.0  1.6 506388 17068 ?Ssl  Mar07   0:45 /usr/bin/docker -d
    root  1330  0.0  0.8 124088  8392 ?Sl   Mar07   0:01 docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 80 -container-ip 192.168.42.2 -container-port 80
    root  9607  0.0  0.0  11720   896 pts/2S+   16:50   0:00 grep --color=auto docker

```

这样看来，docker deamon 正在运行，但是报此错误实属不应该。那么将其停止，再启动。

```

    # service docker stop
    # ps aux | grep docker
    root  9624  0.0  0.0  11716   636 pts/2S+   16:52   0:00 grep --color=auto docker
    # docker -d
    INFO[0000] +job serveapi(unix:///var/run/docker.sock)   
    INFO[0000] Listening for HTTP on unix (/var/run/docker.sock) 
    INFO[0000] +job init_networkdriver()
    INFO[0000] -job init_networkdriver() = OK (0)   
    INFO[0000] WARNING: Your kernel does not support cgroup swap limit. 
    INFO[0000] Loading containers: start.   
    ..........
    INFO[0001] Loading containers: done.
    INFO[0001] docker daemon: 1.5.0 a8a31ef; execdriver: native-0.2; graphdriver: aufs 
    INFO[0001] +job acceptconnections() 
    INFO[0001] -job acceptconnections() = OK (0)  

```

好吧，最有趣的事情是，之前我在 root下一切运行良好，但是现在就像见鬼一样遇到上面的问题。上述的工作完成后，仍有问题。最后我使用 sudo 竟然解决了问题。如下：

```

    # sudo docker info
    Containers: 5
    Images: 32
    Storage Driver: aufs
     Root Dir: /var/lib/docker/aufs
     Backing Filesystem: extfs
     Dirs: 42
    Execution Driver: native-0.2
    Kernel Version: 3.13.0-32-generic
    Operating System: Ubuntu 14.04.1 LTS

```

版权声明：本文为博主原创文章，未经博主允许不得转载。