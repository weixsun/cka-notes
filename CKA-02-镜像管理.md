## 镜像管理 ##
保存镜像
```shell
$ docker images
docker.io/nginx                                       latest              f35646e83998        2 weeks ago         133 MB
...
# 或者docker save f35646e83998 > nginx.tar
$ docker save f35646e83998 -o nginx.tar
$ ll
总用量 133860
-rw-------. 1 root root      1474 10月 18 2018 anaconda-ks.cfg
-rw-------  1 root root 137063424 10月 30 00:17 nginx.tar
-rwxr-xr-x  1 root root       552 10月 26 23:25 set.sh
```

但是再次导入镜像的时候，发现问题
```shell
[root@vms31 ~]# docker rmi f35646e83998
Untagged: docker.io/nginx:latest
Untagged: docker.io/nginx@sha256:ed7f815851b5299f616220a63edac69a4cc200e7f536a56e421988da82e44ed8
Deleted: sha256:f35646e83998b844c3f067e5a2cff84cdf0967627031aeda3042d78996b68d35
Deleted: sha256:9ae13393c37dce86ebd3ea923033503f2cb8f4d6b28fb554827c518a2d171535
Deleted: sha256:423bc419c558f70051d849a661a7a287b61af2037c4ce24f7bbe433e9fb63f39
Deleted: sha256:4cd04e685e3a8e5697bb91e2e6c6b477bc8c4f9a43f05578af3c0a788f011756
Deleted: sha256:611e1562bc2f489d72961d8e2e37f3097d64d9c5212a68c26aab2ad971c98f6d
Deleted: sha256:d0fe97fa8b8cefdffcef1d62b65aba51a6c87b6679628a2b50fc6a7a579f764c
[root@vms31 ~]# docker load -i nginx.tar 
d0fe97fa8b8c: Loading layer [==================================================>] 72.49 MB/72.49 MB
f14cffae5c1a: Loading layer [==================================================>] 64.53 MB/64.53 MB
280ddd108a0a: Loading layer [==================================================>] 3.072 kB/3.072 kB
fe08d9d9f185: Loading layer [==================================================>] 4.096 kB/4.096 kB
cdd1d8ebeb06: Loading layer [==================================================>] 3.584 kB/3.584 kB
Loaded image ID: sha256:f35646e83998b844c3f067e5a2cff84cdf0967627031aeda3042d78996b68d35
[root@vms31 ~]# docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
<none>                                                <none>              f35646e83998        2 weeks ago         133 MB
registry.cn-hangzhou.aliyuncs.com/mysql5-7/mysql5-7   5.7                 0164c13b662c        2 years ago         372 MB
hub.c.163.com/library/centos                          latest              328edcd84f1b        3 years ago         193 MB
hub.c.163.com/library/mysql                           latest              9e64176cd8a2        3 years ago         407 MB
```

说明导出镜像的时候不能使用`IMAGE ID`导出

正确使用方法：
```shell
$ docker save docker.io/nginx > nginx.tar
# 或者
$ docker save docker.io/nginx -o nginx.tar
```

导入镜像：
```shell
$ docker load -i nginx.tar
```

导出全部镜像(不能使用追加的方式`>>`)：
```shell
$ docker save xxx yyy zzz > all.tar
```

导入全部镜像：
```shell
$ docker load -i all.tar
```

查看镜像的Dockerfile
```shell
$ docker history xxxx 显示缩略内容
$ docker history xxxx --no-trunc 可以显示完整的内容
```