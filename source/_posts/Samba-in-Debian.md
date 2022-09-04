---
title: Samba in Debian
date: 2022-09-03 17:39:00
categories: 
  - Server
tags:
  - Debian
  - Samba
  - Linux
  - Nas
---

Samba 实现了 Windows 网络共享，在低功耗 Linux 设备启动 Samba 可把该设备当成一个 Nas 来使用，当然，成品 Nas 肯定是自带 Samba 服务的，无需手动配置。本文介绍 Samba软件的简单配置。

## 下载 Samba

一条命令下载即可，Debian 系使用 apt 命令，Rh 系使用 yum 命令

```bash
sudo apt update && sudo apt install samba
```

下载过程中可能会提示配置一个 DHCP 的问题，建议将设备配置为静态 IP，然后将该选项设置为否

## 编辑配置文件

Samba 的配置文件位于 /etc/samba/smb.conf，我们编辑他

```bash
sudo vim /etc/samba/smb.conf
```

我们可以看见文件大概分为两个部分—— Global Settings 和 Share Definitions。

这里不动Global Settings，我们将Share Definitions下的内容尽数删除，接下来我们要编辑自己的配置

将我提供的配置模板按自己的情况修改之后粘贴进配置文件，保存退出

```bash
# 中括号里是网络地址后接的URL。Win下如此访问：\\IP地址\MyNas
[MyNAS]
   # 共享名称
   comment = Nas
   # 可见性
   browseable = yes
   # 共享文件夹，修改为自己的文件夹
   path = /home/
   # 新建文件权限，不懂别动
   create mask = 0744
   # 新建文件夹权限，不懂别动
   directory mask = 0744
   # 有效用户
   valid users = xxxxxxx
   # 所属用户
   force user = xxxxxxx
   # 所属组
   force group = xxxxxxx
   # 匿名登录
   public = no
   # 可读
   available = yes
   # 可写
   writable = yes
```

修改完成后保存退出，之后测试配置文件是否正常

```bash
sudo testparm
```

显示 Loaded services file OK. 字样即使测试正常，可进行下一步

## 编辑 Samba 用户

### 系统中有该用户，该用户可正常登录

此时正常使用命令将该用户添加为 Samba 用户即可

```bash
## 将 useras 更改为自己配置文件里写的用户
sudo smbpasswd -a users
```

### 系统中无该用户

无该用户时先使用 Linux 自带的用户管理添加该用户，之后回到 3.1 继续即可

## 启动Samba

```bash
## 启动 Samba 命令
sudo /etc/init.d/smbd start
## 停止 Samba 命令
sudo /etc/init.d/smbd stop
## 重启 Samba 命令
sudo /etc/init.d/smbd restart
```

## Windows 下连接 Samba 服务

双击计算机，点击三个点，选择映射网络驱动器

![image-20211008195522011](https://i.loli.net/2021/10/08/a8HiPxr2QbylLCA.png)

输入地址，填写账号密码即可映射为本地磁盘，速度跟内网质量有关

![image-20211008195723372](https://i.loli.net/2021/10/08/eAgOYhsDFjqT3W8.png)
