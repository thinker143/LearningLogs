# 使用Docker安装MySQL

```
命令：
    docker run -d \
      --name mysql \
      -p 3306:3306 \
      -e TZ=Asia/Shanghai \
      -e MYSQL_PASSWORD=123 \
      mysql
```

