# [官方库](https://hub.docker.com/r/library/php/)

- 下载
```
docker pull php:7.2.1-fpm
```
 
- 安装
```
docker run -d -p 9000:9000 --name php-7.2.1-fpm \
-v /data/www:/data/www \
-v /configs:/usr/local/etc/php \
-v /logs:/phplogs \
php:7.2.1-fpm
```

-  安装扩展
```
进入容器 docker exec -it 4b7 /bin/bash

docker-php-ext-install  pdo_mysql
docker-php-ext-install  mysqli
```
 
-  与mysql容器通信
```
docker run --link mysql56 -e MYSQL_ROOT_PASSWORD=123456 -p 9000:9000 --name php-7.2.1-fpm -v /data/www:/data/www -v /configs:/usr/local/etc/php -v /logs:/phplogs -d php:7.2.1-fpm

mysql56 是 mysql容器启动时的命名 --name
```
   

#  配合nginx使用

----

- 获取IP
```
docker inspect php:7.2.1-fpm |grep '"IPAddress"'
```

- 进入nginx容器
```
docker exec -it 9dc90b44b319 /bin/bash
```

```
    location ~ \.php$ {
    #    root           html;
        fastcgi_pass   172.17.0.2:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /data/www/test/$fastcgi_script_name;
        include        fastcgi_params;
    }

```

- 重启后访问
```
docker ps -a

docker restart 9dc90b44b319

http://18.216.248.82/index.php

```

# 测试数据库连接
```php

echo date("Y-m-d H:i:s")."\n";

try {
    $conn = new PDO('mysql:host=mysql56;port=3306;dbname=mysql;charset=utf8', 'root', '123456');
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}

echo 'conn success'."\n";

```
