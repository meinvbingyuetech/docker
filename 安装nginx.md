```
docker pull registry.docker-cn.com/library/nginx:1.13.7

docker images

mkdir -p /data/www &
mkdir -p /data/nginx/conf &
mkdir -p /data/nginx/logs &
touch /data/nginx/conf/nginx.conf

docker run -p 80:80 -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -d nginx:1.13.7

docker run -it -p 80:80 nginx:1.13.7

```

----

```
这个还不行，稍后再试试

docker run -p 80:80 --name nginx1.13.7 -v /data/www:/www -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v data/nginx/logs:/wwwlogs  -d registry.docker-cn.com/library/nginx:1.13.7
```
 
```
做个记录

官方安装：

docker pull nginx
启动跑个静态网页

docker run --name my-nginx -d -p 80:80 -v /webroot:/usr/share/nginx/html:ro -d nginx
这儿简单介绍下ro,默认容器对这个目录有可读写权限，可以通过指定ro，将权限改为只读

添加日志记录

docker run --name my-nginx -d -p 80:80 -v /webroot:/usr/share/nginx/html:ro -v /logs:/var/log/nginx -d nginx
拷贝容器内的配置文件到本地，进行修改等操作

docker cp nginx:/etc/nginx/nginx.conf /config/nginx.conf
重新指定配置文件

docker run --name my-nginx -d -p 80:80 -v /webroot:/usr/share/nginx/html:ro -v /logs:/var/log/nginx -v /config/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
查看运行的容器

docker ps
c4bfc0e2a5aa
重启容器

docker restart c4bfc0e2a5aa
停止容器

docker stop c4bfc0e2a5aa
启动容器

docker start c4bfc0e2a5aa
进入容器

sudo docker exec -it c4bfc0e2a5aa /bin/bash
查看nginx目录

cd /etc/nginx/
```
