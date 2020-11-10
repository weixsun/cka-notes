安装docker
```shell
$ yum install docker -y
```

设置docker 开机自启动并且现在也立马启动
```shell
$ systemctl enable docker --now
```

搜索镜像
```shell
$ docker search nginx
INDEX       NAME                                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker.io   docker.io/nginx                              Official build of Nginx.                        13930     [OK]       
docker.io   docker.io/jwilder/nginx-proxy                Automated Nginx reverse proxy for docker c...   1901                 [OK]
docker.io   docker.io/richarvey/nginx-php-fpm            Container running Nginx + PHP-FPM capable ...   791                  [OK]
docker.io   docker.io/linuxserver/nginx                  An Nginx container, brought to you by Linu...   128                  
docker.io   docker.io/jc21/nginx-proxy-manager           Docker container for managing Nginx proxy ...   103                  
docker.io   docker.io/tiangolo/nginx-rtmp                Docker image with Nginx using the nginx-rt...   99                   [OK]
```
默认是到docker hub 官方仓库搜索镜像的

配置加速器--使用阿里云的加速器
```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://i94yi6it.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```