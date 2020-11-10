## 数据卷的使用

在不使用docker数据卷的情况下，容器的磁盘和宿主机的映射会在容器删除后自动删除（容器stop、restart等不会受到影响）

使用docker 数据卷挂载容器磁盘到宿主机能够永久保存数据。

```shell
$ docker run -dit --restart=always -v c_path 镜像名 命令
```
`c_path` 是容器目录，若不存在则新建.

```shell
$ docker run -dit --restart=always -v h_path:c_path:rw 镜像名 命令
```
`h_path` 是宿主机目录，`c_path` 是容器目录，`rw` 对容器而言点挂载权限（默认就是rw，可选项还有ro）