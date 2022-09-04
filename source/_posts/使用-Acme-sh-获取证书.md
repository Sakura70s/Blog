---
title: 使用 Acme.sh 获取证书
date: 2022-09-04 18:58:27
categories:
  - Server
tags:
  - Acme.sh
  - CloudFlare
  - Debian
  - SSL
  - DNS
---

## 安装 Acme.sh

安装很简单，一行命令搞定

```bash
# my@example.com 替换成自己的邮箱
curl  https://get.acme.sh | sh -s email=my@example.com
```

## 生成证书

Acme.sh 实现了 acme 协议支持的所有验证协议. 一般有两种方式验证: http 和 dns 验证

需要自动话部署的话推荐使用 dns 验证

### 使用 CloudFlare API 进行 DNS 验证

需要去 CloudFlare 拿到 API 密钥

```bash
export CF_Key="sdfsdfsdfljlbjkljlkjsdfoiwje"

export CF_Email="xxxx@sss.com"

# 如果需要泛域名的话，直接再加一个 -d *.xxx.com 就行
acme.sh  --issue  -d example.com  --dns dns_cf
```

## 部署证书

### 部署到 Nginx

需要先再 Nginx 里边配置好证书路径，这里把证书安装到该位置即可

```bash
acme.sh --install-cert -d xxx.com \
--key-file       /cert/private.key  \
--fullchain-file /cert/fullchain.cer \
--reloadcmd     "service nginx force-reload"
```

这样就部署完成了，Acme.sh 会每隔两个月自动尝试重新申请证书
