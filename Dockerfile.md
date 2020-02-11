### 创建了文件后执行命令 
	* 默认文件名 Dockerfile
	* 也可以是 web.dockerfile 、 app.dockerfile 使用时要指定文件名
	```
	docker build -t nginx:test.v3 .

	docker build -t cnpm/vue-cli .

	docker build -t alpine-bash .
	docker run -it --rm alpine-bash


	 docker build -t myphp5606:v1 .
	 docker exec -it myphp5606 sh
	```

- demo-1
	```
	FROM nginx
	RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
	```

- demo-1.1
	```
	# 基础镜像
	FROM alpine
	 
	# 作者信息
	MAINTAINER NGINX Docker Maintainers "1024331014@qq.com"
	 
	# 修改源
	RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
	    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
	 
	# 安装需要的软件
	RUN apk update && \
	    apk add --no-cache ca-certificates && \
	    apk add --no-cache curl bash tree tzdata && \
	    cp -rf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
	 
	# 设置变量 
	ENV NGINX_VERSION 1.14.0

	# 省略安装NG过程.....  

	# 将目录下的文件copy到镜像中
	COPY nginx.conf /etc/nginx/nginx.conf
	COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf
	 
	# 将dist文件中的内容复制到 /usr/share/nginx/html/ 这个目录下面
	COPY dist/  /usr/share/nginx/html/ 
	 
	# 将启动命令搞成个脚本通过脚本启动
	RUN echo "/usr/sbin/nginx" >>/etc/start.sh
	 
	# 开放80端口
	EXPOSE 80
	 
	STOPSIGNAL SIGTERM
	 
	# 启动nginx命令
	CMD ["/bin/sh","/etc/start.sh"]
	```
	
- demo-1.2
	```
	FROM node:6.10.3-slim

	# 安装NG
	RUN apt-get update \    
		&& apt-get install -y nginx

	# 指定工作目录	
	WORKDIR /app

	# 将当前目录下所有文件拷贝到工作目录下
	COPY . /app/
	EXPOSE 80
	RUN  npm install \     
	&& npm run build \     
	&& cp -r dist/* /var/www/html \     
	&& rm -rf /app
	
	# 以前台方式启动NG	
	CMD ["nginx","-g","daemon off;"]
	```

- demo-2
	```
	FROM php:7.0-fpm
	ADD $PWD/php/conf /usr/local/etc/php/conf.d
	RUN /usr/local/bin/docker-php-ext-install pdo_mysql
	```

- demo-3
	```
	FROM php:7.1.22-fpm

	# Update packages
	RUN apt-get update

	# Install PHP and composer dependencies
	RUN apt-get install -qq git curl libmcrypt-dev libjpeg-dev libpng-dev libfreetype6-dev libbz2-dev

	# Clear out the local repository of retrieved package files
	# RUN apt-get clean

	# Install needed extensions
	# Here you can install any other extension that you need during the test and deployment process
	RUN apt-get clean; docker-php-ext-install pdo pdo_mysql mcrypt zip gd pcntl opcache bcmath


	# Installs Composer to easily manage your PHP dependencies.
	RUN curl --silent --show-error https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

	# Install Node
	RUN apt-get update &&\
	    apt-get install -y --no-install-recommends gnupg &&\
	    curl -sL https://deb.nodesource.com/setup_10.x | bash - &&\
	    apt-get update &&\
	    apt-get install -y --no-install-recommends nodejs &&\
	    npm config set registry https://registry.npm.taobao.org --global &&\
	    npm install --global gulp-cli

	CMD php-fpm
	```

- demo-4
	```
	#指定node的镜像源
	FROM node:slim

	MAINTAINER cherics <cherics@163.com>
	#安装淘宝镜像的cnpm
	RUN npm install -g cnpm --registry=https://registry.npm.taobao.org
	#安装vue-cli
	RUN cnpm install -g vue-cli 
	 
	RUN mkdir /code
	COPY . /code
	 
	WORKDIR /code
	#对外开放8080端口
	EXPOSE 8080
	```

- demo-5
	```
	FROM daocloud.io/php:5.6-apache

	# APT 自动安装 PHP 相关的依赖包,如需其他依赖包在此添加
	RUN apt-get update \
	    && apt-get install -y \
		libmcrypt-dev \
		libz-dev \
		git \
		wget \
		libpcre3-dev \

	    # 官方 PHP 镜像内置命令，安装 PHP 依赖
	    # 如果依赖包需要配置参数则通过 docker-php-ext-configure  命令
	    && docker-php-ext-install \
		mcrypt \
		mbstring \
		pdo_mysql \
		zip \


	    # 用完包管理器后安排打扫卫生可以显著的减少镜像大小
	    && apt-get clean \
	    && apt-get autoclean \
	    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \

	    # 安装 Composer，此物是 PHP 用来管理依赖关系的工具
	    && curl -sS https://getcomposer.org/installer \
		| php -- --install-dir=/usr/local/bin --filename=composer

	# 开启 URL 重写模块
	# 配置默认放置 App 的目录
	# Apache 默认的文档目录为 /var/www/html ，将 /app 定义为 Laravel 应用目录，而 Laravel 的项目入口文件位于 /app/public 。为了方便管理，把 /var/www/html  软链接到 /app/public
	RUN a2enmod rewrite \
	    && mkdir -p /app \
	    && rm -fr /var/www/html \
	    && ln -s /app/public /var/www/html

	WORKDIR /app

	# 预先加载 Composer 包依赖，优化 Docker 构建镜像的速度
	# 1.复制 composer.json  和 composer.lock  到 /app，composer.lock  会锁定 Composer 加载的依赖包版本号，防止由于第三方依赖包的版本不同导致的应用运行错误
	# 2.--no-autoloader  为了阻止 composer install  之后进行的自动加载，防止由于代码不全导致的自动加载报错
	# 3.--no-scripts  为了阻止 composer install  运行时 composer.json  所定义的脚本，同样是防止代码不全导致的加载错误问题
	COPY ./composer.json /app/
	COPY ./composer.lock /app/
	RUN composer install  --no-autoloader --no-scripts

	# 复制代码到 App 目录
	COPY . /app

	# 执行 Composer 自动加载和相关脚本
	# 修改目录权限
	RUN composer install \
	    && chown -R www-data:www-data /app \
	    && chmod -R 0777 /app/storage
	```

------------------------------------------------------

```
FROM alpine:3.7

MAINTAINER Rethink 
#更新Alpine的软件源为国内（清华大学）的站点。
RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.4/main/" > /etc/apk/repositories

RUN apk update \
        && apk upgrade \
        && apk add --no-cache bash \
        bash-doc \
        bash-completion \
        && rm -rf /var/cache/apk/* \
        && /bin/bash
```


---------------------------------------------------

```
FROM php:5.6.37-fpm-alpine3.7
 RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
 && apk update\
 && apk add --no-cache libmcrypt-dev freetype-dev libjpeg-turbo-dev \
         git \
         # libfreetype6-dev \
         # libjpeg62-turbo-dev \
         libpng-dev \
 && docker-php-ext-install mcrypt mysqli pdo pdo_mysql mbstring bcmath zip opcache\
 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \
 && docker-php-ext-install -j$(nproc) gd
 ```

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```
	 * requirements.txt
	```
	redis
	flask
	```

```
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
```
 ---------------------------------------------------

 ```
 # 基础镜像
FROM alpine
 
# 作者信息
MAINTAINER NGINX Docker Maintainers "1024331014@qq.com"
 
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
 
# 安装需要的软件
RUN apk update && \
    apk add --no-cache ca-certificates && \
    apk add --no-cache curl bash tree tzdata && \
    cp -rf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
 
# 设置变量 
ENV NGINX_VERSION 1.14.0
 
# 编译安装nginx
RUN GPG_KEYS=B0F4253373F8F6F510D42178520A9993A1C052F8 \
    && CONFIG="\
        --prefix=/etc/nginx \
        --sbin-path=/usr/sbin/nginx \
        --modules-path=/usr/lib/nginx/modules \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/run/nginx.lock \
        --http-client-body-temp-path=/var/cache/nginx/client_temp \
        --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
        --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
        --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
        --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
        --user=nginx \
        --group=nginx \
        --with-http_ssl_module \
        --with-http_realip_module \
        --with-http_addition_module \
        --with-http_sub_module \
        --with-http_dav_module \
        --with-http_flv_module \
        --with-http_mp4_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_random_index_module \
        --with-http_secure_link_module \
        --with-http_stub_status_module \
        --with-http_auth_request_module \
        --with-http_xslt_module=dynamic \
        --with-http_image_filter_module=dynamic \
        --with-http_geoip_module=dynamic \
        --with-threads \
        --with-stream \
        --with-stream_ssl_module \
        --with-stream_ssl_preread_module \
        --with-stream_realip_module \
        --with-stream_geoip_module=dynamic \
        --with-http_slice_module \
        --with-mail \
        --with-mail_ssl_module \
        --with-compat \
        --with-file-aio \
        --with-http_v2_module \
    " \
    && addgroup -S nginx \
    && adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
    && apk add --no-cache --virtual .build-deps \
        gcc \
        libc-dev \
        make \
        openssl-dev \
        pcre-dev \
        zlib-dev \
        linux-headers \
        curl \
        gnupg \
        libxslt-dev \
        gd-dev \
        geoip-dev \
    && curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
    && curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc  -o nginx.tar.gz.asc \
    && export GNUPGHOME="$(mktemp -d)" \
    && found=''; \
    for server in \
        ha.pool.sks-keyservers.net \
        hkp://keyserver.ubuntu.com:80 \
        hkp://p80.pool.sks-keyservers.net:80 \
        pgp.mit.edu \
    ; do \
        echo "Fetching GPG key $GPG_KEYS from $server"; \
        gpg --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$GPG_KEYS" && found=yes && break; \
    done; \
    test -z "$found" && echo >&2 "error: failed to fetch GPG key $GPG_KEYS" && exit 1; \
    gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
    && rm -r "$GNUPGHOME" nginx.tar.gz.asc \
    && mkdir -p /usr/src \
    && tar -zxC /usr/src -f nginx.tar.gz \
    && rm nginx.tar.gz \
    && cd /usr/src/nginx-$NGINX_VERSION \
    && ./configure $CONFIG --with-debug \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && mv objs/nginx objs/nginx-debug \
    && mv objs/ngx_http_xslt_filter_module.so objs/ngx_http_xslt_filter_module-debug.so \
    && mv objs/ngx_http_image_filter_module.so objs/ngx_http_image_filter_module-debug.so \
    && mv objs/ngx_http_geoip_module.so objs/ngx_http_geoip_module-debug.so \
    && mv objs/ngx_stream_geoip_module.so objs/ngx_stream_geoip_module-debug.so \
    && ./configure $CONFIG \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && make install \
    && rm -rf /etc/nginx/html/ \
    && mkdir /etc/nginx/conf.d/ \
    && mkdir -p /usr/share/nginx/html/ \
    && install -m644 html/index.html /usr/share/nginx/html/ \
    && install -m644 html/50x.html /usr/share/nginx/html/ \
    && install -m755 objs/nginx-debug /usr/sbin/nginx-debug \
    && install -m755 objs/ngx_http_xslt_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_xslt_filter_module-debug.so \
    && install -m755 objs/ngx_http_image_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_image_filter_module-debug.so \
    && install -m755 objs/ngx_http_geoip_module-debug.so /usr/lib/nginx/modules/ngx_http_geoip_module-debug.so \
    && install -m755 objs/ngx_stream_geoip_module-debug.so /usr/lib/nginx/modules/ngx_stream_geoip_module-debug.so \
    && ln -s ../../usr/lib/nginx/modules /etc/nginx/modules \
    && strip /usr/sbin/nginx* \
    && strip /usr/lib/nginx/modules/*.so \
    && rm -rf /usr/src/nginx-$NGINX_VERSION \
    \
    # Bring in gettext so we can get `envsubst`, then throw
    # the rest away. To do this, we need to install `gettext`
    # then move `envsubst` out of the way so `gettext` can
    # be deleted completely, then move `envsubst` back.
    && apk add --no-cache --virtual .gettext gettext \
    && mv /usr/bin/envsubst /tmp/ \
    \
    && runDeps="$( \
        scanelf --needed --nobanner /usr/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --no-cache --virtual .nginx-rundeps $runDeps \
    && apk del .build-deps \
    && apk del .gettext \
    && mv /tmp/envsubst /usr/local/bin/ \
    \
    # forward request and error logs to docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
 
# 将目录下的文件copy到镜像中
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf
 
# 开放80端口
EXPOSE 80
 
STOPSIGNAL SIGTERM
 
# 启动nginx命令
CMD ["nginx", "-g", "daemon off;"]
 ```

 docker build -t alpine:nginx .

 # 不进行宿主配置文件日志文件挂载
docker run -tid --name nginx  -p 80:80 \
-m 2048m  --memory-swap=2048m  --cpu-shares=256 \
alpine:nginx

# 挂载配置文件和日志
docker run -tid --name nginx  -p 80:80 \
-v /opt/Webs/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /opt/Webs/nginx/logs/:/var/log/nginx \
-m 2048m  --memory-swap=2048m  --cpu-shares=256 \
alpine:nginx
