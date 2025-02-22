---
layout: post
title: "systemd-外围组件"
category: GNU-linux
date: 2025-02-22 08:00:00 +0800
---

## systemd-cryptsetup

cryptsetup具备加密和解密的功能，但是需要用户手动输入密码。

systemd-cryptsetup具备启动时解密的功能，它读取的是`/etc/crypttab`文件，获取密码自动解密。

我们可以通过cryptsetup命令创建一个加密逻辑卷，并挂载到`/root/abc`路径下：

```sh
dd if=/dev/zero of=/root/disk1 count=100 bs=1M
cryptsetup luksFormat /root/disk1
# 使用大写输入复杂的密码
cryptsetup luksOpen /root/disk1 temp
# 将会创建一个/dev/mapper/temp逻辑卷
mkfs.ext4 /dev/mapper/temp
mkdir /root/abc
mount /dev/mapper/temp /root/abc
```

关闭这个加密逻辑卷：
```sh
umount /root/abc
cryptsetup close temp
```

我们可以通过systemd来解密上面创建的加密逻辑卷，在`/etc/crypttab`中添加如下行：

```sh
temp /root/disk1 luks none
```

执行`systemctl daemon-reload`后，systemd会读取`/etc/crypttab`并自动为这个加密逻辑卷生成一个服务：`systemd-cryptsetup@temp.service`，解密逻辑卷：`systemctl start systemd-cryptsetup@temp.service`。

systemd人性化的地方：如果我们挂载了这个加密逻辑卷到`/root/abc`，执行`systemctl stop systemd-cryptsetup@temp.service`命令时，systemd会自动卸载`/root/abc`。

## Tips

### systemd的一些基础的服务是怎么拉起的？如`systemd-tmpfiles-setup.service`

通过`/lib/systemd/system/sysinit.target.wants/`目录创建的依赖关系。