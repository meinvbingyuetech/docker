- 安装docker
```
yum -y install docker

systemctl start docker
systemctl status docker
systemctl stop docker
 
```

----

- 镜像加速
  - 获取阿里云加速地址
    * https://cr.console.aliyun.com/undefined/instances/mirrors?accounttraceid=10730733-93e0-4aa0-8db5-071b26809dc6
  - vim /etc/docker/daemon.json
  ```
  {
      "registry-mirrors": ["https://grifbizc.mirror.aliyuncs.com"]
  }
  ```
   - systemctl daemon-reload
   - systemctl restart docker

----

### 大多数拉取下来容器进去后都是要用apt-get，不是用yum
### 如果命令太长，可以保存到一个shell文件 test.sh 中，然后直接执行 sh test.sh
----


- 拉取镜像
```
docker search mysql         #查询镜像

docker pull nginx           #拉取镜像
docker pull nginx:1.12.2
docker pull php:7.4.1-fpm
```

----

- 运行容器
  * 常用
  ```
  docker run -itd --rm --name app app

  docker run -itd --name redis -p 6379:6379 redis

  docker run -d -P \
      --name web \
      # -v my-vol:/wepapp \
      --mount source=my-vol,target=/webapp \  # 这个效果跟上面那行同等效果
      training/webapp \
      python app.py
  ```
  
  * 启动nginx
  ```
  docker run -t -i -p 80:80 nginx /bin/bash         #注意第一个80指的是宿主机的端口，第二个80指的是容器内的端口
  docker run -t -i -p 80:80 nginx:latest /bin/bash
  docker run -t -i -p 80:80 nginx:1.12.2 /bin/bash

  docker run --name nginx1.13 -d -p 80:80 -v /web:/usr/share/nginx/html:ro -d nginx

  docker run -itd --name nginx \
  -p 18700:80 \
  -v /home/www:/home/www:ro \
  -v /home/configs/nginx/conf.d:/etc/nginx/conf.d:ro \
  -v /home/logs/nginx:/var/log/nginx \
  --link php:php \
  nginx


  ```
  
  * 启动php
  ```
  docker run -itd --name php \
  -p 9000:9000 \
  -v /home/www:/home/www \
  -v /home/configs/php:/usr/local/etc/php \
  -v /home/logs/php:/phplogs \
  --link mysql56 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  php
  ```
  
  ```
  docker run -d  --rm --name fpm \
  --network mynginx --ip 192.138.0.2 \
  -v /home/www:/www \
  php:7.2.4-fpm-alpine3.7   
  ```
  
  ```
  
  docker run -itd --name php5.6 php:5.6
  
  docker run -p 9000:9000 --name  php-fpm-7.1 -d php:7.1-fpm
  
  docker run -itd --name php -v /home/www:/www php
                   容器名 ； 挂载宿主机/home/www目录到容器目录/www ；镜像名

  ```
  
  ```
  # 启动php容器，执行宿主机的一个php文件
  
  docker run -it --rm  \
  -v /home/www/test/:/www/test  \
  php:7.2.4-cli-alpine3.7 \
  php /www/test/index.php
  ```
  
  ```
  docker run -d \
  --name php \
  -p 9000:9000 \					#宿主端口：容器端口
  -v /data/www:/var/www/html \
  -v /configs:/usr/local/etc/php \
  -v /logs:/phplogs \
  php:7.2.1-fpm
  ```

- privileged 参数
   - docker run --privileged=true
```
使用该参数，container内的root拥有真正的root权限。
否则，container内的root只是外部的一个普通用户权限。
privileged启动的容器，可以看到很多host上的设备，并且可以执行mount。
甚至允许你在docker容器中启动docker容器。
```

- 进入容器
```
docker exec -it 4b7 /bin/bash             # 进入容器，默认使用bash
docker exec -it fpm-7.4.1-alpine3.11 sh   # 有些容器要用sh命令

docker exec -it 4b7 php -m        # 运行容器命令

docker exec -it redis /bin/bash
docker exec -it php /bin/bash
docker exec -it php5.6 /bin/bash
                容器名
                
使用nginx官方镜像，进入容器启动：service nginx start

#php容器安装扩展 
docker-php-ext-install  pdo_mysql
docker-php-ext-install  mysqli
```

- 列出容器
```
docker ps           #列出当前所查看特定容器有正在运行的container

docker ps -a        #列出所有container

docker ps c4bfc0e2a5aa  #查看特定容器

```

- 容器管理
```
docker restart Name/ID
docker start Name/ID
docker stop Name/ID
docker kill Name/ID
docker rm Name/ID                     #删除容器


docker kill -s HUP container-name       #如果要重新载入 NGINX 可以使用以下命令发送 HUP 信号到容器


docker stop $(docker ps -a -q)          #停止所有的container

docker rm $(docker ps -a -q)            #删除所有container
```

----

- 删除镜像
```
docker images                                   #本地镜像集合

docker rmi 镜像ID	                              # 删除镜像
docker rmi -f 镜像ID                            # 强制删除镜像
docker rmi nginx:1.13.7  （REPOSITORY + TAG）

docker rmi $(docker images -q)                  # 删除所有image

docker rmi hub.c.163.com/nce2/mysql:5.6

docker rmi registry.docker-cn.com/library/nginx:latest

rm -rf /var/lib/docker                          # 删除所有镜像、容器和组
```


- 其他
```
#获取容器/镜像的元数据。
docker inspect mysql:5.6

# 获取正在运行的容器的 IP
docker inspect 容器ID/容器名 |grep '"IPAddress"'

docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 容器ID/容器名




docker volume ls 				#查看所有数据卷
docker volume create my-vol 	#创建一个数据卷
docker volume inspect my-vol	#查看指定 数据卷 的信息
docker volume rm my-vol         #删除数据卷
docker volume prune				#清理无主的数据卷



docker cp nginx:/etc/nginx/nginx.conf .


# 查看网络
docker network ls
# 创建网络
docker network create --driver=bridge \    	##单机运行 使用
--subnet=192.138.0.0/16 \
mynginx
# 清理没有使用到的网络
docker network prune -f
```

- 提交镜像
```
docker commit 4b7 php-ext

4b7 可以用 docker ps 获取当前的容器ID
php-ext 是你要重新命名的新镜像名称


docker commit -a "meinvbingyue" -m "开发环境：php+composer+git" a404c6c174a2  dev-app:v1

```

- 推送仓库
```

cd /registry/lexin-crm-server

sudo docker login --username=100010352459 ccr.ccs.tencentyun.com

sudo docker build -t ccr.ccs.tencentyun.com/lexin/lexin-crm:0.4.282 . 

sudo docker push ccr.ccs.tencentyun.com/lexin/lexin-crm:0.4.282

---------------------------

docker login ccr.ccs.tencentyun.com/meinvbingyue/test -u 100002595837 -p yuxiyu110380

docker tag app ccr.ccs.tencentyun.com/meinvbingyue/test:app-v1
docker tag dev-app:v1 ccr.ccs.tencentyun.com/meinvbingyue/test:app-v1
		   REPOSITORY:TAG
		   ImageId

docker push ccr.ccs.tencentyun.com/meinvbingyue/test:app-v1
```

- 卸载docker
```
yum -y remove docker-engine.x86_64
```

