
# 宿主机
- mkdir -p /home/wwwroot/test /home/nginx/logs /home/nginx/conf/conf.d

- cd /home/wwwroot/test && vim index.php
```php
phpinfo();
```

----

# 创建容器
- nginx
```
docker pull nginx
docker run --name docker.nginx -p 18700:80 -d \
    -v /home/wwwroot/test:/usr/share/nginx/html:ro \
    -v /home/nginx/conf/conf.d:/etc/nginx/conf.d:ro \
    -v /home/nginx/logs:/var/log/nginx \
    --link docker.php.fpm.7.1:php \
    nginx
```

- php-fpm
```
docker pull php:7.1-fpm
docker run --name docker.php.fpm.7.1 -d \
    -v /home/wwwroot/test:/www \
	php:7.1-fpm
```
