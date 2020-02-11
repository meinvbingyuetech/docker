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
