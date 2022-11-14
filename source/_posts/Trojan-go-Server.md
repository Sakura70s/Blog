---
hide: index
title: Trojan-go Server
date: 2022-09-04 20:16:46
updated: 2022-11-14 21:33:00
categories:
  - Server
tags:
  - Debian
  - Trojan
  - Trojan-go
  - Passwall
  - GFW
  - Nginx
---

<div class="danger">

> 本工具目前已经弃用
> 目前 GFW 已经可以精准识别 Trojan-go 的 TLS over TLS 的流量
> 并针对其进行封禁
> 推荐爬墙协议更换为 `NaïveProxy` 和 `hysteria`

</div>

<div class="warning">

> 安装 Trojan-go 之前请一定要拥有自己的域名，并申请好 SSL 证书
> 一台境外，且可以在国内联通的服务器
> 该域名需要指向该服务器

</div>

## 下载 Trojan-go

前往项目的 [github-releases](https://github.com/p4gefau1t/trojan-go/releases) 页面下载最新版

Debian 可以使用 wget 命令

```bash
wget https://github.com/p4gefau1t/trojan-go/releases/download/v0.10.6/trojan-go-linux-amd64.zip
```

下载完以后运行 `unzip trojan-go-linux-amd64.zip` 进行解压

## Trojan-go 配置

执行`vim server.json`新建一个配置文件, 并粘贴以下内容

```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "Your Password"
    ],
    "ssl": {
        "cert": "/cert/fullchain.cer",
        "key": "/cert/private.key",
        "fallback_port": 84
    }
}
```

<div class="info">

> 说明:
> run_type: 运行模式
> remote_addr: 伪装网页地址
> remote_port: 伪装网页的端口
> fallback_port: 遇到非 Https 的协议，将其代理到该端口处理
> cert&key: SSL 证书存放的位置

</div>

## 伪装页面配置

可以从 GitHub 里找一个静态网站用以伪装，也可以用自己的网站用作伪装
实在不行使用 Nginx 的默认页面也是可以的

### 伪装页面配置文件

执行`sudo vim /etc/nginx/conf.d/fake_site.conf`新建配置文件，并粘贴以下内容

```bash
server {
    listen 80;
    listen [::]:80;
    listen 84 ssl;
    listen [::]:84 ssl;

    # SSL 配置
    ssl_certificate /cert/fullchain.cer;
    ssl_certificate_key /cert/private.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    # 伪装网站的路径
    root /var/www/html/fake;

    index index.php index.html index.htm index.nginx-debian.html;

    # 绑定到服务器的域名
    server_name your.domain;
 
    client_max_body_size 10m;
}
```

### 重启 Nginx

伪装页面配置保存以后重启 Nginx

```bash
systemctl restart nginx
```

## Systemd 配置

执行 `sudo vim /usr/lib/systemd/system/trojan-go.service`新建服务单元，并粘贴以下内容

```conf
[Unit]
Description=trojan-go
After=network.target

[Service]
User=root
Group=root
Type=simple
PIDFile=/root/trojan-go/trojan-go.pid
ExecStart=/root/trojan-go/trojan-go -config "/root/trojan-go/server.json"
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
```

<div class="info">

> 根据自己的实际情况替换 PIDFile 和 ExecStart 的路径

</div>

### 设置开机自启

```bash
# 先重新加载服务配置文件
sudo systemctl daemon-reload
# 启动 Trojan-go
systemctl start trojan-go
# 设置开机自启
systemctl enable trojan-go
# 查看运行状态
systemctl status trojan-go
# 重启
systemctl restart trojan-go
```

## 预防内存溢出，可以考虑设置每日定时重启

执行`crontab -e`编写当前用户的定时任务，粘贴以下内容

```bash
# m h dM M dWeek shell
40 04 * * * systemctl restart trojan-go
```

执行`crontab -l`查看定时任务