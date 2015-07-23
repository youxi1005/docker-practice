#  [Docker 实践 5：搭建 redmine](http://blog.csdn.net/lincyang/article/details/43851509)

[Redmine](http://blog.csdn.net/lincyang/article/details/www.redmine.org) 是一个开源的项目管理系统，它有如下优势让我选择它作为我的项目管理工具。

1. 支持多项目管理
2. 灵活的角色管理
3. 灵活的 issue/bug 跟踪管理
4. 支持甘特图和日历
5. 支持新闻、文档和文件管理，邮件通知等功能
6. 每个项目有自己的 wiki 和论坛，这一点非常棒
7. 与 SCM 系统集成，支持 SVN, CVS, Git, Mercurial, Bazaar and Darcs 等源代码管理工具，这一点同样非常棒


有了 Redmine，让项目经理不用愁管理项目了。

同样，看看官方是否出 docker 镜像或者其他人作好镜像了，我直接用就好了。

```

    $ docker search redmine
    NAME                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    sameersbn/redmine                                                                72                   [OK]

```

这与在 [docker hub 上搜索](https://registry.hub.docker.com/search?q=redmine&searchfield=)是一样的，虽然没有官方的镜像，那我们就选择星星数量多的镜像，sameersbn/redmine 就成了我的选择。

用 fig 直接快速安装，在自己的 docker 目录下新建 redmine 目录，在里面执行：

```

    ~/docker/redmine$ wget https://raw.githubusercontent.com/sameersbn/docker-redmine/master/fig.yml

```

下载的 fig.yml 内容如下：

```

    postgresql:
      image: sameersbn/postgresql:9.1-1
      environment:
        - DB_USER=redmine
        - DB_PASS=phatiphohsukeuwo
        - DB_NAME=redmine_production
    redmine:
      image: sameersbn/redmine:2.6.1
      links:
        - postgresql:postgresql
      environment:
        - DB_USER=redmine
        - DB_PASS=phatiphohsukeuwo
        - DB_NAME=redmine_production
      ports:
        - "10080:80"

```

直接快速启动就可以了。

```

    ~/docker/redmine$ fig up -d

```

```

    $ docker ps
    CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                            NAMES
    5d5d5a983298        sameersbn/redmine:2.6.1         "/app/init app:start   51 minutes ago      Up 51 minutes       443/tcp, 0.0.0.0:10080->80/tcp   redmine_redmine_1         
    c78a212c1503        sameersbn/postgresql:9.1-1      "/start"               About an hour ago   Up About an hour    5432/tcp                         redmine_postgresql_1   

```

浏览器中输入 http://localhost:10080,

管理员帐号是 admin，密码 admin。

参考：  
https://registry.hub.docker.com/u/sameersbn/redmine/


版权声明：本文为博主原创文章，未经博主允许不得转载。愉快玩耍吧！
    

