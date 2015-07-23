#  [Docker 实践 4：搭建 wordpress](http://blog.csdn.net/lincyang/article/details/43850867)

在系列的第一篇文章《 Docker 实践》中已经 search 到并 pull 了官方的 wordpress 镜像，接下来我们还要 search 一个官方的 mysql 将二者结合，搭建一个可用的 wordpress 站点。

首先，搞定 mysql

1.search

```

    $ docker search mysql
    NAME DESCRIPTION STARS OFFICIAL   AUTOMATED
    mysqlMySQL is a widely used, open-source relati...   456   [OK]   

```

2.pull

```

    $ docker pull mysql

```

其次，考虑二者的联合

```

    $ docker images
    REPOSITORY               TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    wordpress                latest              ecc04d6d638c        2 weeks ago         470 MB
    mysql                    latest              aca96d9e6b5c        2 weeks ago         282.7 MB

```

wordpress 启动命令是这样的：

```

    $ sudo docker run --name some-wordpress --link some-mysql:mysql -d wordpress

```

```

    启动 WordPress 容器时可以指定的一些环境参数包括

    -e WORDPRESS_DB_USER=... 缺省为 “root”
    -e WORDPRESS_DB_PASSWORD=... 缺省为连接 mysql 容器的环境变量 MYSQL_ROOT_PASSWORD 的值
    -e WORDPRESS_DB_NAME=... 缺省为 “wordpress”
    -e WORDPRESS_AUTH_KEY=..., -e WORDPRESS_SECURE_AUTH_KEY=..., -e WORDPRESS_LOGGED_IN_KEY=..., -e WORDPRESS_NONCE_KEY=..., -e WORDPRESS_AUTH_SALT=..., -e WORDPRESS_SECURE_AUTH_SALT=..., -e WORDPRESS_LOGGED_IN_SALT=..., -e WORDPRESS_NONCE_SALT=... 缺省为随机 sha1 串

```

针对 wordpress 的启动命令，我们需要这样针对：

1. 给 wordpress 可以起个名字，这个好办
2. --link 参数，这需要我们先启动 mysql，然后将其名字链接上
3. 端口 -p 参数，默认是 80 端口，但是被我占用了，这里我们映射到 8080

启动的 mysql 的命令：

```

    $ docker run --name mysql_wordpress -e MYSQL_ROOT_PASSWORD=wordpress  -d  mysql

```

mysql 的密码，姑且这样暴露着吧。

对应 mysql，wordpress 的启动命令如下：

```

    $ docker run --name docker_wordpress --link mysql_wordpress:mysql -p 8080:80 -d wordpress

```

接下来就可以在浏览器中输入 http://localhost:8080 进行 wordpress 的配置了。

第三，用 fig 来配置

实践证明，用 fig 配置是最好的途径。在上面的基础上，我们只需在自己的 docker 目录下新建目录如 wordpress-docker，再建 fig 配置文件 fig.yml 如下：Enjoy！

```

    wordpress:
        image: "wordpress:latest"
        ports:
            - "8080:80"
        links:
            - db:mysql

    db:
        image: "mysql:latest"
        expose:
            - "3306"
        environment:
            - MYSQL_ROOT_PASSWORD=wordpress

```

每次启动只需执行本目录下的 fig up -d 就可以了！

参考：  
https://github.com/docker-library/wordpress/blob/aee00669e7c43f435f021cb02871bffd63d5677a/Dockerfile  

如果想用 fig 搭建 wordpress，个人感觉更方便一些，参考如下网址：  
http://dockerpool.com/static/books/docker_practice/fig/wordpress.html

版权声明：本文为博主原创文章，未经博主允许不得转载。


