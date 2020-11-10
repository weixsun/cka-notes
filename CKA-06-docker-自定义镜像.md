## 自定义镜像 ##

把容器导出为镜像:
```shell
docker export 容器名 > filename.tar 
```

导入:
```shell
cat filename.tar | docker import - 镜像名:tag
```
> 可能没有CMD命令，自己指定即可。




0. 前置条件：基础镜像
1. 生成一个临时镜像
2. 执行Dockerfile文件中操作
3. 导出为新的镜像
4. 删除这个临时镜像

```Dockerfile
FROM hub.c.163.com/library/centos
MAINTAINER docker
RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY- CentOS-* 
ADD epel.repo /etc/yum.repos.d/ 
RUN yum install -y nginx 
EXPOSE 80 端口2 端口3 
#env
ENV name zhangsan
#user
RUN useradd tom
USER tom
#volume
VOLUME ["/xx","/yy"]
CMD ["nginx", "-g","daemon off;"]
```
`MAINTAINER` 关键词声明镜像作者

`EXPOSE` 关键词只做标记说明使用

`COPY`和`ADD`的意思是一样，都是拷贝的意思，但是 `ADD`带有自动解压功能 `COPY`没有自动解压功能
> 不管`COPY`还算`ADD`都只能拷贝当前目录下文件到容器中

`VOLUME` 容器卷，随机找一个目录映射到`/xx`，随机找一个目录映射到`/yy`,无法指定具体宿主机目录的（符合镜像定义--可移植）。

docker run时可以使用`-u xxx`指定使用哪个用户登录进入容器


```shell
$ docker build -t 镜像名:tag . -f filename 
```
`.`表示到当前目录找`Dockerfile`文件，如果自定义的镜像制作语法不是写在`Dockerfile`中，则必须使用`-f fileName`指定镜像制作语法在哪个具体文件中。


使用nginx 举例，如果在容器中修改了nginx.conf配置文件，此时是不需要使用`systemctl restart nginx` 重启nginx的（即使使用也报错），只需要执行`docker restart nginx`重启镜像即可。


**配置可以ssh的centos镜像**
```Dockerfile
FROM centos:v1 
MAINTAINER docker

#安装软件包
RUN yum install openssh-clients openssh-server -y

#生成key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
RUN ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key

#修改配置文件，提升ssh的速度
RUN sed -i '/UseDNS/cUseDNS no' /etc/ssh/sshd_config

#修改用户密码
RUN echo "root:redhat" | chpasswd

#或者
#RUN echo redhat | passwd --stdin root

EXPOSE 22 
CMD ["/usr/sbin/sshd", "-D"]
```
