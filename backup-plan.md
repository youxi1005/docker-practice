# Docker 实践 9：备份方案

## 1 两个文件系统

先提一下两个重要的文件系统概念，一个是 aufs，一个是 vfs.

aufs 是一个类似于 Unionfs 的可堆叠联合文件系统。它将多个目录整合成单一的目录。ubuntu 对其有良好的支持，因此 docker 的镜像就存储在 aufs 文件系统下。

vfs 是 linux 的内核中一个重要概念，这个虚拟文件系统可以让 open()、read()、write()等系统调用不用关心底层的存储介质和文件系统类型就可以工作的粘合层。

## 2 docker 镜像与容器的存储

![picture9.1](images/9.1.jpg)

docker 的层次结构如上图。 

docker 安装在/var/lib/docker 目录，来这里访问需要 root 权限。

```

    /var/lib/docker# du -sh *
     1.8G    aufs
    80K containers
    36K execdriver
    452K    graph
    15M init
     8.0K    linkgraph.db
     4.0K    repositories-aufs
     240M    tmp
    8.0K    trust
    203M    vfs
    28K volumes

```


从文件夹大小就可以看出 aufs 和 vfs 是实际存东西的，在最开始的时候，我的只把重点放在了 aufs 上。

当前有两个 docker 镜像，使用 docker-compose 部署的 mediawiki。

```

    # docker images
    REPOSITORY               TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    mysql                    5.7.5               c171a2256c3b        3 weeks ago         322.8 MB
    nickstenning/mediawiki   latest              d32d732341b0        16 months ago       619.5 MB

```

与之对应的 repositories-aufs 如下：

```

    /var/lib/docker# cat repositories-aufs | python -mjson.tool
    {
        "Repositories": {
            "mysql": {
                "5.7.5": "c171a2256c3b9d13c77b2b3773eaaad95a9b4a8735613ce46722f79fa29fa366"
            },
            "nickstenning/mediawiki": {
                "latest": "d32d732341b07cfc87006d09ea75514d8c10774dfbc83c74034ac28b1491557b"
            }
        }
    }

```

趁着 tree 参数可用，我们来看看 images 的层级结构：

```

    # docker images -tree
    Warning: '-tree' is deprecated, it will be removed soon. See usage.
    ├─511136ea3c5a Virtual Size: 0 B
    │ └─36669626e49c Virtual Size: 84.98 MB
    │   └─d5570ef1464a Virtual Size: 84.98 MB
    │     └─170ff2cfa64a Virtual Size: 85.3 MB
    │       └─ef7c16d26957 Virtual Size: 116.7 MB
    │         └─3749ee37c2a6 Virtual Size: 116.8 MB
    │           └─5431d6b9bc80 Virtual Size: 116.8 MB
    │             └─0dc7d8240120 Virtual Size: 116.8 MB
    │               └─a37379b8d1ce Virtual Size: 116.8 MB
    │                 └─cad6baf3ee5b Virtual Size: 322.8 MB
    │                   └─facdf8305638 Virtual Size: 322.8 MB
    │                     └─a10cdfaf13cf Virtual Size: 322.8 MB
    │                       └─444b88bd8367 Virtual Size: 322.8 MB
    │                         └─83dfd5776ff8 Virtual Size: 322.8 MB
    │                           └─7d0b9c0b29d2 Virtual Size: 322.8 MB
    │                             └─c171a2256c3b Virtual Size: 322.8 MB Tags: mysql:5.7.5
    └─8dbd9e392a96 Virtual Size: 128 MB
     └─be647841828f Virtual Size: 128 MB
         └─7f06ad8f23b1 Virtual Size: 272.5 MB
          └─9fc1b767f7ff Virtual Size: 383.1 MB
            └─46af893ffb5f Virtual Size: 439.2 MB
              └─00a79ad7ea4a Virtual Size: 439.2 MB
                └─69f7b013792e Virtual Size: 439.2 MB
                  └─854a5bd5be72 Virtual Size: 439.2 MB
                    └─c219ca4d0d53 Virtual Size: 439.2 MB
                      └─42c69a7e8ad3 Virtual Size: 439.2 MB
                        └─09652ed5f7ef Virtual Size: 459.6 MB
                          └─814a97e8dc1c Virtual Size: 539.6 MB
                            └─a7bc400b42c2 Virtual Size: 539.6 MB
                              └─68013db1eb6d Virtual Size: 619.5 MB
                                └─ca839611a34f Virtual Size: 619.5 MB
                                  └─63e9d31943ea Virtual Size: 619.5 MB
                                    └─1b95e1c4efd9 Virtual Size: 619.5 MB
                                      └─9df1e07fdf78 Virtual Size: 619.5 MB
                                        └─3e8e8966dcfb Virtual Size: 619.5 MB
                                          └─3e0c3340e16e Virtual Size: 619.5 MB
                                            └─d32d732341b0 Virtual Size: 619.5 MB Tags: nickstenning/mediawiki:latest

```

再去看看 graph 目录：

 
```

    /var/lib/docker/graph# ls
    00a79ad7ea4a77bac24386226563b86ee92db49073e6417ce10dbe175777b7ff  814a97e8dc1c5b86b3a4a21e2c64abac2e69ad1214f084c60e78b7b9b4d91e75
    09652ed5f7efeb035c7f29e7fe16b600ff4066362ea95f678974d7940f89f021  83dfd5776ff844e112fa504701370e7a0e20ecdf43afc75cf9018f05c6e46699
    0dc7d82401208db703d519db581d7dddb39a090e2bf0dd6b5a64dcf9a743e6aa  854a5bd5be7280e1a53d451e3806aaf202a4cfffdf1701cf10cd1036bccb6bdc
    170ff2cfa64adbd757c0dad0c1cb4bbb61e11301a16cd8f29b40cb3217ec7254  8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c
    1b95e1c4efd993116df499fa9951b865ec99e8a455427cffc9c61364bc3c5295  9df1e07fdf780918f8fafaba0f5470d1c6357232617e42b6d190383fd0710e37
    36669626e49c623f3b2aea41052bbc9de0506807d92137bf76c0fd3264eaff34  9fc1b767f7ff6d935efd6e897882d202fcbd041a154c400b595b9e0f1b2092dc
    3749ee37c2a6df4df9f720c7c07ecc5af695a2a5853bac1f8ba6450ddc310343  a10cdfaf13cf8dbaa07798f29265ec37a5bec4dc5f2b6f8c449edb2658b0a571
    3e0c3340e16e991af227912b27441b392762476f15d6749c3c1333fab63c5927  a37379b8d1ce392404d50d97a02afb92bd774a45191b3dc98c512fa6af394335
    3e8e8966dcfba21708e772ec165edcb617061028572e279decf7b374f4f3a866  a7bc400b42c28673642376884b7a8a93c76548198ebb4c4993a65cdfaf81ccd2
    42c69a7e8ad3ba37bfddeaf52ea54381d487311ba5c75d6a91811abeaad8438b  be647841828f846812df2968e24aa9db46c1a2033c0c60577e2b91fdcf7d4c09
    444b88bd8367b8fc2d023b5ceb77cec3784963bbfc03d1d30f96b97a22cfbc44  c171a2256c3b9d13c77b2b3773eaaad95a9b4a8735613ce46722f79fa29fa366
    46af893ffb5fe47b7de23a3f2f8972dbf51e78a0327ca46b7848c1cdfd48f97d  c219ca4d0d536c79955628e17aa8d6e6b6a61bc50c486bcd304a70a790985627
    511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158  ca839611a34f4eb30e582694d9a1cdabf11d96b4c36bf69c1a82fb069ba0509a
    5431d6b9bc80429667488287e8189cc4d4b14791e43e45b374e132e340fb15db  cad6baf3ee5b2798947cdcfbdd3bf4c38c3f094bfbec876fb27da426528107fa
    63e9d31943ea386e435763524d9ea3a8cb253747113abbb53191121b815572da  d32d732341b07cfc87006d09ea75514d8c10774dfbc83c74034ac28b1491557b
    68013db1eb6d12803db7cbb52e95e6f29c8141c095b8cb50d245925d7f70d774  d5570ef1464a43fe282dd2705b38a2d739812b0b8036a49cfa09811737cfbed4
    69f7b013792e1f9c63176a7e348cb38277182f8bcf3d1d6d2c26ff9c4885b4ec  ef7c16d26957dc5dea9ad23433c7dd6400ab0a1a5cf461a080312dfaea50eeee
    7d0b9c0b29d271a8be6ca9252da15ae4f01dd386ac2b452563232aaaddd5eaf9  facdf830563864336b80233167577315520bca3fbbd86d8f995e40f0e81e238c
    7f06ad8f23b10227423823e3acc7e368c012f41a30660c3fbad661a9a4f8fe67  _tmp

```


长 id 与 tree 的短 id 是对应的，这样就明晰了。在上文中给 container 传文件用到了神奇通道，其实就是 aufs 中的 mnt。


## 备份方案

第一种情况，挂载 host 目录的，比如之前的 hg-server。备份就很简单，直接把那些 hg 源码目录在 host 做备份即可。

第二种情况，需要在 container 中安装新软件。用 export 或者 save 命令吧。

```

    sudo docker commit 9ab6e234c9ba linc-wiki
    
    sudo docker images REPOSITORY   TAG IMAGE IDCREATED VIRTUAL SIZE linc-wikilatest  b5a1e34b01c214 seconds ago  689.7 MB
    
    sudo docker export 9ab6e234c9ba > /home/linc/docker/images-bk/linc-wiki-export.tar
    sudo docker save linc-wiki > ../images-bk/linc-wiki-save.tar
    
    $ du -sh *
    495M    linc-wiki-export.tar
    672M      linc-wiki-save.tar
    
    sudo cat /home/linc/docker/images-bk/linc-wiki-export.tar | sudo docker import - docker_hgweb
    sudo docker load --input ../images-bk/linc-wiki-save.tar

```

第三种情况，mysql 中的数据哪里去啦？ 

这个问题困扰我几天了。就拿部署的 mediawiki 说吧，没有将数据文件链接到 host 中，而是放到了/var/lib/mysql中，如下：

```

     /var/lib/mysql# ls
    auto.cnf  ib_logfile0  ib_logfile1  ibdata1  ibtmp1  my_wiki  mysql  performance_schema

```


我嘗試着新设置 volume 不过没有成功，也做了其他努力，都失败了。最后一次大搜索中，发现这些数据文件被放在了 vfs 中。

```

    /var/lib/docker/vfs# tree -L 3
    .
    └── dir
        └── cb4012594631102fc8a69aa6cb4a9aa2dd2be3d39d2564c94ae91ac5e3eb4aec
            ├── auto.cnf
            ├── ibdata1
            ├── ib_logfile0
            ├── ib_logfile1
            ├── ibtmp1
            ├── mysql
            ├── my_wiki
            └── performance_schema

    5 directories, 5 files

```

尝试了将其放入我的 ubuntu server 虚拟机中的 docker 相同目录下，mediawiki 用 host 中 save 后的镜像，我的 mediawiki 在 ubuntu server 虚拟机中完美复活！ 

也就是说，我只需要备份 host 系统中 docker 下的 vfs 的数据就可以了。问题解决，可以肆意在 wiki 中添加重要内容而不担心被毁掉了。

参考：   

[http://www.programfish.com/blog/?p=9 ](http://www.programfish.com/blog/?p=9)  

[http://blog.csdn.net/junjun16818/article/details/38423391 ](http://blog.csdn.net/junjun16818/article/details/38423391)  

gitlab 的维护：[http://www.tuicool.com/articles/bYbi2mJ](http://www.tuicool.com/articles/bYbi2mJ)

版权声明：本文为博主原创文章，未经博主允许不得转载。
