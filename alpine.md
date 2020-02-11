- 修改镜像源
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

- 可以直接运行命令
docker run alpine echo '123'

- 添加软件
apk add --no-cache composer
apk add --no-cache git


echo "http://dl-4.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
apk --update add --no-cache <package>


---------------------------
