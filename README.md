```
docker search mysql

docker pull nginx
docker pull nginx:1.12.2

本地镜像集合
docker images		
```

- 删除镜像
```
docker images

docker rmi 镜像ID	        删除镜像
docker rmi -f 镜像ID      强制删除镜像
docker rmi nginx:1.13.7  （REPOSITORY + TAG）

docker rmi $(docker images -q)  删除所有image

docker rmi hub.c.163.com/nce2/mysql:5.6

docker rmi registry.docker-cn.com/library/nginx:latest
```

```
列出当前所有正在运行的container
docker ps
列出所有container
docker ps -a
查看特定容器
docker ps c4bfc0e2a5aa

```

```
docker restart Name/ID
docker start Name/ID
docker stop Name/ID
docker kill Name/ID
docker rm Name/ID

停止所有的container
docker stop $(docker ps -a -q)
删除所有container
docker rm $(docker ps -a -q)
```

```
systemctl status docker
systemctl stop docker
systemctl start docker 
```

```
docker rmi ed9c93747fe1   删除image

## 上面的命令不会删除镜像、容器，卷组和用户自配置文件。
## 删除所有镜像、容器和组

rm -rf /var/lib/docker
```

```
卸载docker
 
yum -y remove docker-engine.x86_64
```

```
docker inspect 容器ID或容器名 |grep '"IPAddress"'
```

```
docker run -t -i -p 80:80 nginx /bin/bash
docker run -t -i -p 80:80 nginx:latest /bin/bash
docker run -t -i -p 80:80 nginx:1.12.2 /bin/bash
```

```
docker run --name nginx1.13 -d -p 80:80 -v /web:/usr/share/nginx/html:ro -d nginx

docker run -p 9000:9000 --name  php-fpm-7.1 -d php:7.1-fpm
```

```
运行 容器命令
docker exec -it 4b7 php -m
 
进入 容器
docker exec -it 4b7 /bin/bash
```
 
```
docker commit 4b7 php-ext

4b7 可以用 docker ps 获取当前的容器ID
php-ext 是你要重新命名的新镜像名称
```
