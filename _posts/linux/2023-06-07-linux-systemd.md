---
layout:       post
title:        "systemd：应知应会"
author:       "licunlong"
header-style: text
catalog:      true
tags:
    - systemd
    - linux
---

# systemd

## udevd

**udevadm trigger** 命令只会触发change事件，而不是触发所有事件。需要出发特定类型的事件时，例如add事件，执行`udevadm trigger --action=add`。