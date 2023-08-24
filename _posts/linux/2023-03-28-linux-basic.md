---
layout:       post
title:        "linux零散的配置和命令"
author:       "licunlong"
header-style: text
catalog:      true
tags:
    - linux
    - basic
---

## 安装openEuler 22.03 虚拟机

**注意:20.03版本的虚拟机貌似不好启用网卡相关的配置.需要使用openEuler 22.03版本.**

**虚拟网卡的类型需要使用NAT模式,网桥模式一般的笔记本都不支持.**

**注意在虚拟机配置nameserver,在虚拟机配置GATEWAY**

```shell
# /etc/resolv.conf
nameserver 192.168.122.1
```

```shell
# /etc/sysconfig/network-scripts/ifcfg-ens7
GATEWAY=192.168.122.1
```

## 安装CentOS6.0虚拟机

其实网卡已经添加好了，通过`ip addr`能够查看到，需要做的是让它开机自启即可。直接修改`/etc/sysconfig/network-scripts/ifcfg-eth0`文件。

## docker 使用

[docker使用指南](https://yeasy.gitbook.io/docker_practice/)

```shell
docker run -it -v /sys/fs/cgroup:/sys/fs/cgroup:rw -v /docker-run:/run:rw c7-systemd /sbin/init
docker run -it -v /docker-run:/run:rw docker-test /sbin/init
```

```shell
docker import oesystemd.tar.gz oesystemd:1
```

```shell
docker export ID -o oesystemd.tar.gz
```

```shell
docker load -i openeuler-22.03-lts:latest.tar.gz
```

### Dockerfile将systemd作为docket的一号进程

```shell
FROM openeuler-22.03-lts:latest
RUN yum install -y systemd
CMD ["/sbin/init"]
```
`docker build -t docker-systemd .`执行这个命令构建容器镜像。


## 关于锁屏

* 熄屏时间: gsettings set org.gnome.desktop.session idle-delay 1800
* 熄屏后是否锁定: gsettings set org.gnome.desktop.screensaver lock-enabled true
* 熄屏后到锁定的等待时间: gsettings set org.gnome.desktop.screensaver lock-delay 3600

## 关于普通用户的登陆shell

修改/etc/passwd的配置将/bin/sh修改为/bin/bash

## 关于标题栏

debian的标题栏太宽，将标题栏的宽度缩减到24个像素。

```css
/* .config/gtk-3.0/gtk.css */
headerbar {
    min-height: 16px;
    padding-left: 2px; /* same as childrens vertical margins for nicer proportions */
    padding-right: 2px;
    margin: 0px; /* same as headerbar side padding for nicer proportions */
    padding: 0px;
}
```

## NetworkManager

NetworkManager自动根据`/etc/sysconfig/net-scripts/`目录下的配置来配置网卡。需要注意的是，这里的配置文件是用户自己写的，可能有冗余，也可能跟内核实际的设备有差异，因此无法保证这里的配置能够正常生效。

**内核观测到的网卡设备，或者说实际的网卡设备：** 可以通过`ifconfig -a`获取到。

**udev可能会根据udev规则修改网卡的名称，修改后可能导致/etc/sysconfig/net-scripts/目录下的配置因为匹配不到网卡名无法生效。**

**dhclient只是会向dhserver获取申请动态IP。**

**eth0** 名字的网卡是内核的名字。<https://www.cnblogs.com/yinfutao/p/9634350.html>

## debian的intel网卡驱动安装

搜索firmware-iwlwifi下载安装。<https://packages.debian.org/search?keywords=firmware-iwlwifi>

## grub字体太小

修改grub的分辨率：/etc/default/grub: GRUB_GFXMODE=1280x720。执行`update-grub`使配置生效。

## debian输入法配置

fcitx很多时候是无辜的。为了验证fcitx是否ok，最好是在edge、vscode等非原生gtk应用上。

* select system keyboard layout：这里配置的为底层的快捷键布局，edge的英文输入布局。建议直接用dvorak。选择后会生成一个对应的输入法。
* shuangpin：这里配置的为双拼，可以点击键盘，配置布局，建议直接用qwerty的布局。
* 可以通过添加group的方式给其他用户配置一个pinyin输入法。
* terminal、notes等系统自带应用需要通过修改系统设置里面的language。改成中文，英文输入时是常规布局，改成dvorak,英文输入时是dvorak。

### 自己使用的配置group

必须按照顺序配置如下：

1. Keybord-English(US)-English(Dvorak)
2. Keybord-English(US)-English(Dvorak,alt.intl)
3. Shuangpin （注意需要点击小键盘，切换Language=Any language, Layout=English(Us), Variant=Default）

### 常规布局配置group

* Keyboard-English(US)
* Keyboard-Chinese
* Pinyin

### global options:

* 使用`ctrl+space`作为快捷键：（1）Enumerate Input Method Forward，注意钩上“Skip first input method while enumerating”。（2）Activate Input Method。**注意：删掉“Trigger Input Method”的快捷键**。
* 使用`ctrl+shift+space`作为切换group的快捷键：（1）Enumerate Input Method Group Forward。

这样后续就可以直接通过`ctrl + space`，切换中英文输入法；使用`ctrl+shift+space`，切换到常规布局group，而且切换到常规布局后，快捷键的布局也会一并切换。

## 其他博主的一些优秀案例

<https://qileq.com/article/202201270002/>
