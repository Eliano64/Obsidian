---
title: Nginx
tags:
  - Nginx
categories:
  - Base Knowledge
date: 2025-11-29 00:00:00
katex: true
---

# 1. 问题情景

在aliyun服务器（ubuntu）上部署了一个后端服务，端口为 8080。然而，由于安全策略的限制，外部无法直接`<ip>:8080`访问该端口。

# 2. 解决方案

配置 Nginx 作为反向代理，将外部请求转发到该后端服务即可。

## 2.1. 配置Nginx

1. 安装Nginx

```sh
sudo apt update
sudo apt install nginx
```

## 2.2. 反向代理

编辑Nginx配置文件

```sh
sudo vim /etc/nginx/sites-available/default
```
在 `server` 块中添加以下配置：

```nginx
location /api/ {
    proxy_pass http://localhost:8080/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

保存并退出编辑器, 然后重启Nginx

```sh
# 检查配置是否正确
sudo nginx -t
# 重启Nginx
sudo systemctl restart nginx
```

这样配置后，外部就可以通过 `<ip>/api/` 访问到后端服务了。实质是Nginx将外部请求`<ip>:80/api/` 转发到服务器的 `localhost:8080` 上。

# 3. 其他相关配置

## 3.1. 服务器采用SSL

使用 certbot 自动申请 Let's Encrypt 证书：

```sh
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d <your-domain>
```

按提示操作，certbot 会自动修改 Nginx 配置以启用 HTTPS（监听 443 端口），并复用你之前定义的转发规则（如 `/api/` 转发到 8080）。

## 3.2. 将React前端打包后静态托管

通常在本地或 CI 环境执行构建，然后将产物部署到服务器。

执行构建命令 `npm run build`，然后将生成的 `dist` 目录下的所有文件上传到服务器的 `/var/www/html` 目录。

```sh
# 先清空服务器目标目录里旧的静态文件
ssh user@<server-ip> "rm -rf /var/www/html/*"
```

```sh
# 使用 scp 上传
scp -r dist/* user@<server-ip>:/var/www/html/
```

然后在 Nginx 配置中, 添加以下 location 块来托管这些静态文件：

```nginx
location / {
    root /var/www/html;
    index index.html;
    try_files $uri $uri/ /index.html;
}
```

然后重启Nginx

```sh
sudo nginx -t
sudo systemctl restart nginx
```
