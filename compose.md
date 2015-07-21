#  Docker 实践 8：Compose

今天要在我的本子上搭建一个 mediawiki 环境，之前的经验，用 fig 去配置是最简单的了。可是下载 fig 失败，去官网一看才知道，fig 已经被 compose 工具取代了。原文是这样说的：

> Fig has been replaced by Docker Compose, and is now deprecated. The new documentation is on the Docker website.

既然如此，就去官网看看 compose 到底为何物。 

compose 是用来在 docker 中定义和运行复杂应用的小工具，比如在一个文件中定义多个容器，只用一行命令就可以让一切就绪并运行。它的功能与我们所熟知的 fig 相似，换句话说，compose 是 fig 的替代产品，fig 就这样退出 docker 的历史舞台了。

然而在 github 上的 compose 有这样的说法：

> Fig has been renamed to Docker Compose, or just Compose for short. This has several implications for you:

> The command you type is now docker-compose, not fig.
You should rename your fig.yml to docker-compose.yml.

看来 fig 是被重命名成 compose 了，配置文件变成了 docker-compose.yml，其他都几乎一样。不但 fig 不能下载了，原来有 fig 工具的环境用 fig 去搭建 mediawiki 都不可用了，报错如下：

```

    fig up -d
    Creating hgserver_mediawiki_1...
    Pulling image amclain/hgweb...
    Traceback (most recent call last):
      File "<string>", line 3, in <module>
      File "/code/build/fig/out00-PYZ.pyz/fig.cli.main", line 31, in main
      File "/code/build/fig/out00-PYZ.pyz/fig.cli.docopt_command", line 21, in sys_dispatch
      File "/code/build/fig/out00-PYZ.pyz/fig.cli.command", line 28, in dispatch
      File "/code/build/fig/out00-PYZ.pyz/fig.cli.docopt_command", line 24, in dispatch
    ...
    fig.progress_stream.StreamOutputError: Get https://index.docker.io/v1/repositories/amclain/hgweb/images: dial tcp: lookup index.docker.io on 10.202.72.118:53: read udp 10.202.72.118:53: i/o timeout

```

如此看来，使用 compose 是必须的了。 

下面说说 compose 的用法。 

1.安装 compose 

OS X 和 64 位的 Linux 用如下命令安装。

```

     # curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

    #chmod +x /usr/local/bin/docker-compose

```

其他平台可以像 python 包一样安装：

```

     $ sudo pip install -U docker-compose

```

2.命令简介

    $ docker-compose 
    Fast, isolated development environments using Docker.

    Usage:
      docker-compose [options] [COMMAND] [ARGS...]
      docker-compose -h|--help

    Options:
      --verbose                 Show more output
      --version                 Print version and exit
      -f, --file FILE           Specify an alternate compose file (default: docker-compose.yml)
      -p, --project-name NAME   Specify an alternate project name (default: directory name)

    Commands:
      build     Build or rebuild services
      help      Get help on a command
      kill      Kill containers
      logs      View output from containers
      port      Print the public port for a port binding
      ps        List containers
      pull      Pulls service images
      rm        Remove stopped containers
      run       Run a one-off command
      scale     Set number of containers for a service
      start     Start services
      stop      Stop services
      restart   Restart services
      up        Create and start containers

3.compose 编写 mediawiki 的 docker-compose.yml 

首先编写 compose 的配置文件，语法与 fig 类似，文件名为 docker-compose.yml，内容如下：

```

    wiki2:
        image: 'nickstenning/mediawiki'
        ports:
            - "8880:80"
        links:
            - db:database
        volumes:
            - /data/wiki2:/data

    db:
        image: "mysql"
        expose:
            - "3306"
        environment:
            - MYSQL_ROOT_PASSWORD=defaultpass

```

4.创建并启动 mediawiki

```

    $ docker-compose up -d

```

版权声明：本文为博主原创文章，未经博主允许不得转载。