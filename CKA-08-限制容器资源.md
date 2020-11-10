## 限制容器资源 ##
Linux 本身的一种安全特性--Cgroup

对内存的限制
```shell
/etc/systemd/system/memload.service.d
cat 00-aa.conf
[Service]
MemoryLimit=512M
```

对CPU亲和性限制
```shell
ps mo pid,comm,psr $(pgrep httpd) /etc/systemd/system/httpd.service.d
cat 00-aa.conf
[Service]
CPUAffinity=0
```

### 对容器内存的限制 ###
```shell
$ docker run -it --name=c1 -m 200M --restart=always hub.c.163.com/library/centos

# 容器中执行：
$ rpm -ivh /mnt/memload-7.0-1.r29766.x86_64.rpm
$ memload 1024

$ docker stats db
CONTAINER           CPU %               MEM USAGE / LIMIT       MEM %               NET I/O             BLOCK I/O           PIDS
db                  0.03%               196.8 MiB / 3.843 GiB   5.00%               1.94 kB / 648 B     58.5 MB / 12.8 MB   27
```
`-m 200M`：限制内存使用200M，如果超出规定的内存报如下错误提示：

<img src="https://raw.githubusercontent.com/weixsun/cka-notes/main/images/docker_container_oom.png">

`docker stats` 查看容器资源消耗情况


### 对容器CPU的限制 ###


```shell
$ cat /dev/zero > /dev/null &
```














