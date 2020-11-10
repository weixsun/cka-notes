
## docker 网络管理 ##
只讲解单节点的网络设置，不涉及跨节点网络设置，跨节点网络设置由容器编排章节说明。

在容器内部能够ping 公网是因为docker 开启了`端口转发`，查看docker是否开启端口转发：
```shell
$ cat /proc/sys/net/ipv4/ip_forward
1
```
> 1 代表开启了端口转发（默认docker安装完毕后开启）；0 代表未开启端口转发功能，此时容器内部是不能通公网的。

查看docker 全部网卡列表：
```shell
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e4d9456f6e93        bridge              bridge              local
dce4f1a728b4        host                host                local
7d24b990158c        none                null                local
```

查看docker 某网卡信息：
```shell
$ docker network inspect bridge
```

```shell
$ man -k docker
...
docker-network-connect (1) - connect a container to a network
docker-network-create (1) - create a new network
docker-network-disconnect (1) - disconnect a container from a network
docker-network-inspect (1) - inspect a network
docker-network-ls (1) - list networks
docker-network-rm (1) - remove one or more networks
...
```

新建docker 网络
```shell
$ docker network create -d bridge --subnet=10.0.0.0/24 mynet
54517a21af362f2973b36d252fa249f5e0727567c033bb8c288d395d9f1fabb1

$ docker network list
NETWORK ID          NAME                DRIVER              SCOPE
e4d9456f6e93        bridge              bridge              local
dce4f1a728b4        host                host                local
54517a21af36        mynet               bridge              local
7d24b990158c        none                null                local
```
`-d` = `--driver`
`NAME`：表示交换机名称

`DRIVER`：表示驱动类型
- bridge 桥接模式：容器内通公网
- host 仅主机模式：完全复制使用了宿主机网络信息（感觉上不是容器启动的似的，容器内通公网、宿主机）
- null : 容器中没有网卡

`SCOPE`：local 表示本地生效，如果是跨主机可以设置为global


容器启动指定网络：
```shell
$ docker run -dit --name=dbother --restart=always --network=mynet -e MYSQL_ROOT_PASSWORD=redhat -e MYSQL_USER=Tom -e MYSQL_PASSWORD=redhat hub.c.163.com/library/mysql
56f10abc82df0d74395e84d279decc4a312b50abeb4486514f223c22e407f011

$ docker inspect dbother | grep -i 'ipaddress'
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "10.0.0.2",

$ ip a
...
6: br-54517a21af36: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:c1:0a:23:68 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1/24 scope global br-54517a21af36
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c1ff:fe0a:2368/64 scope link 
       valid_lft forever preferred_lft forever
...
```
`--network`默认值为`bridge`

此时dbother 容器内通公网、10.0.0.1、172.17.0.1，不通172.17.0.2（db容器，处于不同网络）

容器端口映射：
```shell
docker run -dit --name=dbother --restart=always -p h_port:c_port -e MYSQL_ROOT_PASSWORD=redhat -e MYSQL_USER=Tom -e MYSQL_PASSWORD=redhat hub.c.163.com/library/mysql
```
`h_port` 是宿主机端口，`c_path` 是容器端口


容器互联

lab-搭建一个wordpress博客
```shell
docker run -dit --name db --restart=always -e MYSQL_ROOT_PASSWORD=redhat -e MYSQL_DATABASE=wordpress mysql 
```
```shell
docker run -dit --name blog --restart=always -e WORDPRESS_DB_HOST=172.17.0.2 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=redhat -e WORDPRESS_DB_NAME=wordpress -p 80:80 wordpress 
```
> 不推荐，直接使用ip 会有ip冲突占用问题。

或者
```shell
$ docker run -dit --name=blog --restart=always --link=db:mysql -v /blog:/var/www/html -p 80:80 hub.c.163.com/library/wordpress
```
`--link=db:mysql` 中db表示某容器名称，mysql是db的别名，容器中但凡再使用到db容器时，一律使用别名。
