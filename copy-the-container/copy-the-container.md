# [Docker 实践 7：容器与主机拷贝数据](http://blog.csdn.net/lincyang/article/details/44176569)

在 [Docker 实践 2](http://blog.csdn.net/lincyang/article/details/43450999) 中使用-v 参数将主机与容器中相关目录联系在一起（挂载），所以我们可以用这个通道将想要互相拷贝的数据放入其中，这样就可以用 cp 命令来复制文件了。

除了这个办法，我们还可以分别用不同的命令来拷贝数据。

## 从容器中像主机拷贝数据

docker 提供了 cp 命令，用法如下：

```

    # docker ps
    CONTAINER IDIMAGE COMMANDCREATED STATUS  PORTS NAMES
    a77a72ac178ctutum/apache-php:latest   "/run.sh"  21 hours agoUp 21 hours 0.0.0.0:8080->80/tcp  phpapache_phpapache_1
    # docker-enter a77a72ac178c
    root@a77a72ac178c:~# ls /var/www/html
    index.php  logo.png
    root@a77a72ac178c:~# exit
    logout
    
    # docker cp a77a72ac178c:/var/www/html /var/www/
    # ls /var/www/
    app  download  index.html
    # ls /var/www/app/
    index.php  logo.png

```

## 从主机向容器中拷贝数据

这里要使用一个 docker 提供的神奇通道来完成主机向容器的数据传输。 

首先要用 docker inspect 方法获得容器的完整 id，

```

    inspect   Return low-level information on a container

```

然后用/var/lib/docker/aufs/mnt/通道来完成拷贝。 

举例如下：

```

    # docker inspect -f '{{.Id}}' a77a72ac178c
    a77a72ac178c1e35708d2af446197c10239b0b1bd8932104578e334b83eb93a2
    # cp docker/docker-start.sh /var/lib/docker/aufs/mnt/a77a72ac178c1e35708d2af446197c10239b0b1bd8932104578e334b83eb93a2/root/
    # docker-enter a77a72ac178c
    # pwd
    /root
    # ls
    docker-start.sh

```

版权声明：本文为博主原创文章，未经博主允许不得转载。