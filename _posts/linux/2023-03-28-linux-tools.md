---
layout:       post
title:        "linux的一些工具配置"
author:       "licunlong"
header-style: text
catalog:      true
tags:
    - basic
    - linux
---

## samba

在debian配置samba：

1. 安装：apt install samba
2. 写配置文件：

```shell
# /etc/samba/smb.conf
[share]
   comment = einsler share
   path = /home/einsler
```

3. 添加用户：
    3.1 添加系统用户：useradd USERNAME
    3.2 将系统用户添加到samba：smbpasswd -a USERNAME
    3.3 在samba中启用这个用户：smbpasswd -e USERNAME

4. 测试是否ok：`smbclient //localhost/share -U einsler`。注意这里的share是步骤2中方括号里面配置的值。

**注意点：** 如果客户端测试ok,但是网络连接不上，就要考虑防火墙的问题了！腾讯云官方屏蔽了一些端口，需要我们将端口修改为自定义端口。

修改端口号为8445：

```shell
# /etc/samba/smb.conf
smb ports = 8445
```
