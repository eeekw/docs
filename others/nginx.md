# Nginx

## 配置文件

默认配置文件`nginx.config`

默认配置文件目录`/usr/local/nginx/conf`，`/etc/nginx`或`/usr/local/etc/nginx`

## 命令行

`nginx -s stop`停止nginx进程

`nginx -s quit`等待当前请求完成后停止nginx进程

`nginx -s reload`重新加载配置文件

`nginx -s reopen`重新开始日志文件

`kill -s QUIT 1628`发送QUIT信号给指定进程ID的进程

`ps -ax | grep nginx`列出所有运行中的nginx进程

## 配置文件结构

`http`与`event`指令位于主上下文中，`server`位于`http`中，`location`位于`server`中

```text
http {
    server {
    }
}
```

### web server

```text
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```

匹配文件扩展名

```text
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

### proxy server

```text
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```

### `location`匹配规则

```text
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

首先按序匹配正则表达式，如果满足，则不再继续匹配。如果没有满足的正则表达式，则匹配前缀最长的`location`

