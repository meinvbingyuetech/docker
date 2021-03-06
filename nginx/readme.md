# [官方库](https://hub.docker.com/r/library/nginx/)

- 下载
```
docker pull nginx:1.12.2
```
- 查看本地images
```
docker images
```

- 运行启动
  *  <a href="#默认配置文件">查看nginx默认配置文件</a>
```
# 指定网站目录
docker run --name nginx1.12.2 -p 80:80 -v /data/www:/data/www:ro -d nginx:1.12.2

# 指定日志
docker run --name nginx1.12.2 -p 80:80 -v /data/www:/data/www:ro -v /logs:/var/log/nginx -d nginx:1.12.2

# 执行配置文件 （采用这个，要先创建 /configs/nginx 并创建 default.conf 文件）
docker run -d --name nginx -p 80:80 \
-v /home/www:/data/www:ro \
-v /home/configs/nginx:/etc/nginx/conf.d:ro  \
-v /home/logs:/var/log/nginx \
nginx:1.13

/data/www:/data/www:ro -> ro 十分重要，意思是只读权限，如果不加，则会启动不成功
 
目录那些会自动创建，不用手动建
 
在 宿主机和nginx容器内都创建 /data/www 的目录 下面就放网站目录 e.g: /data/www/test
```
    

- 查看容器状态
```
docker ps -a
```

- 进入容器
```
docker exec -it nginx1.12.2 /bin/bash
```

```
docker run -it -p 80:80 nginx:1.12.2 /bin/bash
service nginx start
 
这种方式如果命令行(exit)退出则服务也将停止
```

- 进入容器后，修改nginx的网站配置
```
apt-get update
apt-get install -y vim
vim /etc/nginx/conf.d/default.conf
```

```
server {
    location / {
        root   /data/www/test;        // <------------ 这里
        index  index.html index.htm;
    }

    location ~ \.php$ {
    #    root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /data/www/test/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

- 退出终端，创建访问的静态文件
```
exit;

mkdir -p /data/www/test

vim /data/www/test/index.html
```
 
- 重启服务，并访问
```
docker ps -a

docker restart 4e4

http://IP/
```
 
----
 
```
 
下载：
docker pull nginx

启动跑个静态网页，这儿简单介绍下ro,默认容器对这个目录有可读写权限，可以通过指定ro，将权限改为只读
docker run --name my-nginx -p 80:80 -v /webroot:/usr/share/nginx/html:ro -d nginx

添加日志记录
docker run --name my-nginx -p 80:80 -v /webroot:/usr/share/nginx/html:ro -v /logs:/var/log/nginx -d nginx

拷贝容器内的配置文件到本地，进行修改等操作
docker cp nginx:/etc/nginx/nginx.conf /config/nginx.conf
docker cp nginx1.12.2:/etc/nginx/nginx.conf /config/nginx.conf         （REPOSITORY + TAG）

重新指定配置文件(先停掉服务)
docker run --name my-nginx -d -p 80:80 -v /webroot:/usr/share/nginx/html:ro -v /logs:/var/log/nginx -v /config/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx

查看运行的容器
docker ps c4bfc0e2a5aa

重启容器
docker restart c4bfc0e2a5aa

停止容器
docker stop c4bfc0e2a5aa

启动容器
docker start c4bfc0e2a5aa

进入容器
docker exec -it c4bfc0e2a5aa /bin/bash

查看nginx目录
cd /etc/nginx/
```

----
<a name="默认配置文件"></a>
```nginx
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /data/www/test;
        index  x.html;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
    #    root           html;
        fastcgi_pass   172.17.0.2:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /data/www/test/$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```
