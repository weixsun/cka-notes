## 容器管理 ##
查看正在运行状态的容器
```shell
$ docker ps
```

查看所有状态的容器
```shell
$ docker ps --all
```


容器的生命周期
```shell
$ docker run --name=c1 hub.c.163.com/library/centos

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ docker ps -a
CONTAINER ID        IMAGE                          COMMAND             CREATED             STATUS                     PORTS               NAMES
5359cf53007c        hub.c.163.com/library/centos   "/bin/bash"         10 seconds ago      Exited (0) 9 seconds ago                       c1

```
`--name=c1` : 为容器指定名称，否则使用随机名称(随机字符串)


运行centos镜像发现其并不是预期的 `running` 状态，而是`Exited` 状态，这是由于centos的CMD启动命令导致的。

```shell
docker history hub.c.163.com/library/centos
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
328edcd84f1b        3 years ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0 B                 
<missing>           3 years ago         /bin/sh -c #(nop)  LABEL name=CentOS Base ...   0 B                 
<missing>           3 years ago         /bin/sh -c #(nop) ADD file:63492ba809361c5...   193 MB
```

`CMD ["/bin/bash"]` 是一次性命令，一次运行后，容器运行结束，所以再看见的状态就是`Exited` 状态了。


```shell
$ docker rm c1
c1

$ docker run -t -i --name=c1 hub.c.163.com/library/centos
[root@8f8c19464144 /]# 
[root@8f8c19464144 /]# 
[root@8f8c19464144 /]# 
[root@8f8c19464144 /]# exit
$
```
`-t` : 分配模拟终端（假的TTY）

`-i` : 交互式的TTY(否则终端卡住)

`exit` : 退出交互式终端，此时容器会被自动stop

```shell
$ docker start c1

$ docker attach c1
[root@8f8c19464144 /]# 
[root@8f8c19464144 /]# 
[root@8f8c19464144 /]# 
[root@8f8c19464144 /]# exit
```

`docker start c1` 启动停止的c1容器

`docker attach c1` 连接到正在运行的容器的终端

发现此时在终端执行`exit`,容器还是会自动stop，所以在运行镜像的时候可以使用`--restart`选项来解决这个问题。
```shell
docker run -t -i --restart=always --name=c1 hub.c.163.com/library/centos
[root@82a957796494 /]# 
[root@82a957796494 /]# 
[root@82a957796494 /]# 
[root@82a957796494 /]# 
```
`--restart=always` : 重启策略为`always`
> docker stop CONTAINER 命令优先级高于`--restart=always`


当运行某个容器的时候，不想立即进入容器的终端，可以使用`-d` 选项解决
```shell
docker run -t -i -d --restart=always --name=c1 hub.c.163.com/library/centos
4d900dfa916a3b7eb382ebf98a2f6459ed7513158021cf2ed397817bb6436e17

```
`-d` : 在后台运行容器并打印容器ID

可以将`-d`、`-i`、`-t`三个选项合并为一`-dit`:
```shell
docker run -dit --restart=always --name=c1 hub.c.163.com/library/centos
```

**显示某个容器的运行进程：**
```shell
$ docker top --help

Usage:	docker top CONTAINER [ps OPTIONS]

Display the running processes of a container

Options:
      --help   Print usage
```
例如：
```shell
$ docker top c1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                6991                6974                0                   16:17               pts/1               00:00:00            /bin/bash
```

运行容器时运行指定命令，而非镜像中内置命令：
```shell
docker run -dit --restart=always --name=c1 hub.c.163.com/library/centos sleep 100
```


**在正在运行的容器中运行命令:**
```shell
$ docker exec --help

Usage:	docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables (default [])
      --help                 Print usage
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
```

例如：
```shell
docker exec -it c1 bash
```

**获取容器的日志：**
```shell
$ docker logs --help

Usage:	docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --help           Print usage
      --since string   Show logs since timestamp
      --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
```
例如：
```shell
$ docker logs -f db
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
error: database is uninitialized and password option is not specified
```

运行一个Mysql数据库容器：
```shell
$ docker run -dit --name=db --restart=always -e MYSQL_ROOT_PASSWORD=redhat -e MYSQL_USER=Tom -e MYSQL_PASSWORD=redhat hub.c.163.com/library/mysql
25619f4762aeff6135fde4222f33e21db8b43b3f9d6ba55a716bce87fd0a0977
```
`-e` : Set environment variables (default [])

> 此时在宿主机中能够查到mysqld进程`ps aux | grep -v grep | grep -E 'mysql'`，说明docker 共享了宿主机的CPU和内存资源。


返回有关Docker对象的低级信息：
```shell
$ docker inspect --help

Usage:	docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
      --help            Print usage
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
```

例如：
```shell
$ docker inspect db
[
    {
        "Id": "25619f4762aeff6135fde4222f33e21db8b43b3f9d6ba55a716bce87fd0a0977",
        "Created": "2020-10-31T08:52:33.46975307Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "mysqld"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 8394,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-10-31T08:52:33.636101381Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:9e64176cd8a206f88336506fe52cd8f87423147dc197d0250175dddc39465e90",
        "ResolvConfPath": "/var/lib/docker/containers/25619f4762aeff6135fde4222f33e21db8b43b3f9d6ba55a716bce87fd0a0977/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/25619f4762aeff6135fde4222f33e21db8b43b3f9d6ba55a716bce87fd0a0977/hostname",
        "HostsPath": "/var/lib/docker/containers/25619f4762aeff6135fde4222f33e21db8b43b3f9d6ba55a716bce87fd0a0977/hosts",
        "LogPath": "",
        "Name": "/db",
        "RestartCount": 0,
        "Driver": "overlay2",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "journald",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "always",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "docker-runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Name": "overlay2",
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/74b61c0c468c48614fc120b8dcf09a00405a966a2d5bad5702bac85801974cac-init/diff:/var/lib/docker/overlay2/230c9756e413002d25f07feb35e93fc3ae8f78d652f418839712d2d94cbf74f3/diff:/var/lib/docker/overlay2/8129c3ee503ac0dae87ae8d62a0a8f3521b93b2d121b6ab26c58287e7dd2f5f0/diff:/var/lib/docker/overlay2/78ae4e350113c955cc0018218c6a20262be55f8e7f120dc659cd18a92f6593d4/diff:/var/lib/docker/overlay2/291b5d5fcfc5846953c685358206b4b4e304f80554fc5e3f94f9e4f23d0a9b23/diff:/var/lib/docker/overlay2/f224f607bb8f82ca6d5032cd5949130e0946b532967338e0c48a00d0659486b3/diff:/var/lib/docker/overlay2/e225ded7d61591ff1d3fad192c79802959927cbf033f3a80cda27f7de5236f77/diff:/var/lib/docker/overlay2/2b4cf3bcb3070cc9d3ef533b4f27390f77653481d834bd9cfd31440d907608e5/diff:/var/lib/docker/overlay2/57ef432825f6de98a80506a2ee653ffa245f617e0d6b52a92442f0c0aa78b525/diff:/var/lib/docker/overlay2/94268d40b00646824a14b963c68f4135ce358a737668f0745ba92a51eb7742af/diff:/var/lib/docker/overlay2/97a4343c208610219a79dcf04ef36bce4261a1bc78869344b3d089c5dd1a4552/diff:/var/lib/docker/overlay2/d5496af7148be191743fe17cfc56700c2541ac8ded9d7d005450945e5eab3cf2/diff",
                "MergedDir": "/var/lib/docker/overlay2/74b61c0c468c48614fc120b8dcf09a00405a966a2d5bad5702bac85801974cac/merged",
                "UpperDir": "/var/lib/docker/overlay2/74b61c0c468c48614fc120b8dcf09a00405a966a2d5bad5702bac85801974cac/diff",
                "WorkDir": "/var/lib/docker/overlay2/74b61c0c468c48614fc120b8dcf09a00405a966a2d5bad5702bac85801974cac/work"
            }
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "f4304f322dad484e6aed6519b2e9f21953061a66865a2a677ea5b8dc97765f06",
                "Source": "/var/lib/docker/volumes/f4304f322dad484e6aed6519b2e9f21953061a66865a2a677ea5b8dc97765f06/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "25619f4762ae",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "MYSQL_ROOT_PASSWORD=redhat",
                "MYSQL_USER=Tom",
                "MYSQL_PASSWORD=redhat",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.7",
                "MYSQL_MAJOR=5.7",
                "MYSQL_VERSION=5.7.18-1debian8"
            ],
            "Cmd": [
                "mysqld"
            ],
            "ArgsEscaped": true,
            "Image": "hub.c.163.com/library/mysql",
            "Volumes": {
                "/var/lib/mysql": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "273883b2a662c415549b44bc234a723c0e950863d68c539cf5cae31e8d96f190",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "3306/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/273883b2a662",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "d3a002b9a5bdcac520a9769df50486c11f1751f87ca0e435437a44fbbff92361",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "3f32e8d07a229adb75d3cb80ef3320eb06331c88b3ace1da1e3017905634d917",
                    "EndpointID": "d3a002b9a5bdcac520a9769df50486c11f1751f87ca0e435437a44fbbff92361",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
        }
    }
]

```


当退出容器时自定删除容器：
```shell
$ docker run -it --rm hub.c.163.com/library/centos /bin/bash
```
> `--rm` 选项和`--restart` 选项是冲突的，不能同时使用。 

指定容器的hostname：
```shell
$ docker run -dit -h node --name=c1 镜像名 命令
```
`-h` 选项可以指定容器的hostname


`docker rm CONTAINER`无法直接删除`running` 态的镜像，有两种办法：

在正在运行的容器中远程执行一个命令：
```shell
# 查看容器/tmp目录文件内容
$ docker exec db ls /tmp

# 查看容器IP 信息
$ docker exec db ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link 
       valid_lft forever preferred_lft forever
```

**在容器和本地文件系统之间复制文件/文件夹：**
```shell
docker cp --help

Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
	docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

Copy files/folders between a container and the local filesystem

Options:
  -a, --archive       Archive mode (copy all uid/gid information)
  -L, --follow-link   Always follow symbol link in SRC_PATH
      --help          Print usage
```

例如：向正在运行的容器中拷贝文件：
```shell
$ docker cp /etc/hosts c1:/tmp
```

例如：将正在运行的容器中的文件拷贝出来：
```shell
$ docker cp c1:/etc/hosts ./
```


**方式一**
```shell
$ docker rm -f c1
```
`-f` : 强制删除正在运行的容器（使用SIGKILL）

**方式二**
先停止镜像，再删除镜像
```shell
$ docker stop CONTAINER
$ docker rm c1
```