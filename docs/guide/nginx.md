---
icon: nginx
title: 反代
copyright: false
order: -2
---

:::info 注意
使用反向代理之前记得要修改默认的 80 端口号哦
:::

### nginx-proxy-manager

强烈推荐 [nginx-proxy-manager](https://nginxproxymanager.com/)这个项目！它可以帮你自动管理反代配置，并申请相应的 `https` 证书。

### caddy

第二推荐的是 [caddy](https://caddyserver.com/)，一个现代的高性能 web 服务器，它也可以自动帮你配置好 `https`

配置文件参考：

```
example.com {
  tls admin@example.com
  proxy / <IP:端口号> {
    transparent
  }
}
```

### nginx

如果你还是想想用 nginx 的话，那好吧。
http 版本的：

```nginx
server {
  gzip on;
  gzip_min_length 1k;
  gzip_comp_level 9;
  gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
  gzip_vary on;
  gzip_disable "MSIE [1-6]\.";
  listen 80 ;
  # 改为你的网址
  server_name example.com;
  proxy_buffers 8 32k;
  proxy_buffer_size 64k;
  location / {
    # 改为容器的 PORT
    proxy_pass http://127.0.0.1:<PORT>;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Upgrade $http_upgrade;
  }
}
```

https 版本的：

```nginx

server {
  listen 80;
  # 改为你的网址
  server_name example.com;
  # 重定向为 https
  return 301 https://$host$request_uri;

}
server {
  listen 443 ssl http2;
  # 改为你的网址
  server_name example.com;
  # 证书的公私钥
  ssl_certificate /path/to/public.crt;
  ssl_certificate_key /path/to/private.key;

  location / {
    # 改为容器的 PORT
    proxy_pass http://127.0.0.1:<PORT>;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Upgrade $http_upgrade;
  }

}
```