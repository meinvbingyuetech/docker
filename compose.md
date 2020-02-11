- 安装
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- 查看最新版本，然后替换上面的1.24.1
```
https://github.com/docker/compose/releases
```

- 添加权限
```
sudo chmod +x /usr/local/bin/docker-compose
```

- 创建软链(可选)
```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

- 查看版本
```
docker-compose --version
docker-compose --help
```

- 启动运用程序
  * 在docker-compose.yml 所在目录中执行
  * 默认文件名就叫做 docker-compose.yml ,如果你写成其他文件名，需要手工指定 （-f） 该配置文件的位置
```
docker-compose up
docker-compose up -d  #后台执行
```

- 停止容器
```
docker-compose stop
```

- 启动未删除的容器
```
docker-compose start
```

- 删除已经停止的容器
   * -f 参数：不要求确认删除
```
docker-compose rm -f
```

---- 

```
version: '3'
services:
  web:
    build: .
    ports:
   - "5000:5000"
    volumes:
   - .:/code
    - logvolume01:/var/log
    links:
   - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

```
version: "3"
services: 
  web1: 
    container_name: web1
    image:"centos:httpd"
    ports: 
      - "8080:80"
    privileged: true
    volumes: 
      - "/home/joefom/nginx/web1/:/var/www/html/"
  web2: 
    container_name: web2
    image: "centos:httpd"
    ports: 
      - "8081:80"
    privileged: true
    volumes: 
      - "/home/joefom/nginx/web2/:/var/www/html/"
```
