## 本地仓库

docker images

## 安装mysql

docker run mysql
 
## 搜索

docker search nginx

## 删除image

docker rmi ed9c93747fe1 

## 上面的命令不会删除镜像、容器，卷组和用户自配置文件。
## 删除所有镜像、容器和组

rm -rf /var/lib/docker

## 卸载docker
 
yum -y remove docker-engine.x86_64
