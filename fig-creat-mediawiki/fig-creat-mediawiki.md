#  [Docker 实践 3：fig 搭建 mediawiki](http://blog.csdn.net/lincyang/article/details/43451043)

fig，无花果。[fig 项目](http://blog.csdn.net/lincyang/article/details/www.fig.sh)源自 docker，可以认为是快速搭建基于 Docker 的隔离开发环境的工具。

## 一、安装 fig

```

    $ mkdir docker; cd docker

    $ curl -L https://github.com/docker/fig/releases/download/1.0.1/fig-`uname -s`-`uname -m` > fig

    $ sudo chmod +x fig; sudo mv fig /usr/local/bin/

```

## 二、搭建 mediawiki

使用个人构建的镜像，github 地址：[https://github.com/bopjiang/wikimedia-docker](https://github.com/bopjiang/wikimedia-docker)

在 docker 目录下执行如下命令：

```

    $ git clone https://github.com/bopjiang/wikimedia-docker.git
    $ cd wikimedia-docker
    $ fig up -d
    
```

此时有两个容器启动：

```

    r$ docker ps
    CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                  NAMES
    21182a060c17        nickstenning/mediawiki:latest   "/usr/bin/mediawiki-   7 hours ago         Up 7 hours          0.0.0.0:8880->80/tcp   wikimediadocker_wiki2_1   
    728ec09c3552        mysql:5.7.5                     "/entrypoint.sh mysq   7 hours ago         Up 7 hours          3306/tcp               wikimediadocker_db_1    

```

## 三、fig.yml

fig.yml 用来配置镜像构建的具体内容，此 wiki 的 fig.yml 在 wikimedia-docker 目录下，内容如下：

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
        image: "mysql:5.7.5"
        expose:
            - "3306"
        environment:
            - MYSQL_ROOT_PASSWORD=defaultpass

```

image：用来指定镜像，如果本地没有，fig 将会尝试去远程 pull 这个镜像。

ports：暴露的端口.

links：在其他服务中连接容器。

volumes: 卷挂载路径，容器中的/data/目录挂载到主机的/data/wiki2下。在 wiki 配置完毕后，将
 LocalSettings.php 文件放置在主机的/data/wiki2 目录下。

expose：也是暴露端口，与 ports 的区别是不发布到宿主机的端口，只被连接的服务访问。

environment：设置环境变量。

## 四、wiki 的配置

浏览器中输入 localhost:8880,首次启动会让进入配置界面。完成后生成 LocalSettings.php 文件。也可以直接在这个配置文件中作配置。

生成的 LocalSettings.php 文件要拷到/data/wiki2目录下（配置文件中定义的卷挂载路径），并增加其 r 属性就可以了。

还记得在 yml 配置文件中数据库主机名是什么吧？database，对了，那么在配置中也要这样填写，如图：

![picture3.1](images/3.1.png)

## 五、wiki 的使用技巧

1.左侧导航栏的配置

以管理员身份登录，在搜索栏中输入 MediaWiki:sidebar

进入配置界面后就可以编辑了。比如：

```

    <pre>
    navigation
        http://192.168.0.111:8880/index.php?title=Category:XXX|XXX
        mainpage|mainpage-description
         portal-url|portal 
    </pre>

```



3.分类

文章的末尾加入"category"标签即可将此文章放到了 xxx 分类中，一篇文章可以加入多个分类。

比如：[[category:XXX]]

4.新文章

在 Search 中输入你的文章名称即可 Edit。

5.换行

用 br 标签可以换行。<br>

空一行也会有换行效果。

6.pre 标签包围源代码

例如：
```

    <pre>
    private int mSize;
    </pre>

```

## 六、保存容器和导入

```

    sudo docker commit 9ab6e234c9ba linc-wiki

    sudo docker images REPOSITORY               TAG                 IMAGE ID            CREATED             VIRTUAL SIZE linc-wiki                latest              b5a1e34b01c2        14 seconds ago      689.7 MB

    sudo docker export 9ab6e234c9ba > /home/linc/docker/images-bk/linc-wiki-export.tar
    sudo docker save linc-wiki > ../images-bk/linc-wiki-save.tar

    $ du -sh *
    495M    linc-wiki-export.tar
    672M    linc-wiki-save.tar

    sudo cat /home/linc/docker/images-bk/linc-wiki-export.tar | sudo docker import - docker_hgweb
    sudo docker load --input ../images-bk/linc-wiki-save.tar

```

**附录：**

1.fig 使用报错及解决

fig running error:

```

    $ fig up
    Couldn't connect to Docker daemon at http:/ - is it running?
    
    If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.

```

fix it:

```

    1) Change the DOCKER_OPTS in /etc/default/docker to:
    DOCKER_OPTS="-H tcp://127.0.0.1:4243 -H unix:///var/run/docker.sock"
    
    2) Restart docker
    sudo restart docker
    
    3) Make sure that docker is running on localhost:4243 
    $ netstat -ant  |grep 4243
    tcp0  0 127.0.0.1:4243  0.0.0.0:*   LISTEN
    
    4) Set DOCKER_HOST (.bashrc)
    export DOCKER_HOST=tcp://localhost:4243
    
    $ echo $DOCKER_HOST
    tcp://localhost:4243 

```

参考：  
dockerpool.com/static/books/docker_practice/fig/yml_ref.html 


版权声明：本文为博主原创文章，未经博主允许不得转载。