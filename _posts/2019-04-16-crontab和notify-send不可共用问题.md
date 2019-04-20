---
layout: post
title: crontab和notify-send不可共用问题
tags: 
- crontab
- linux
- notify-send
---

<!--more-->

因为notify-send是一个GUI程序并且直接与DBUS通信，而crontab执行后台程序所以在crontab中调用是没有效果的。只有设置环境变量以后程序才能联系桌面通知程序的绘画总线地址，从而起作用。

在Mint19或Ubuntu18.04中，只需手动为cron作业提供`DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"`即可。

但该套接字实际上并不存在在Mint 18或Ubuntu 16.04上，但是可以编写脚本实现  
将该脚本命名为cron-notify放在`/usr/bin`下，并给其授权`chmod +x cron-notify`, 当然在Mint19(本机环境)上也能起作用  

```bash
#!/bin/sh
if [ -z "$DBUS_SESSION_BUS_ADDRESS" ]; then 
    if [ -z "$LOGNAME" ]; then
        EUID=$(id -u)
    else
        EUID=$(id -u "$LOGNAME")
    fi
    [ -z $EUID ] && exit

    if [ -S /run/user/$EUID/bus ]; then
        export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$EUID/bus"
    else
        SESSION=$(loginctl -p Display show-user "$LOGNAME" | cut -d= -f2)
        [ -z "$SESSION" ] && exit
        LEADER=$(loginctl -p Leader show-session "$SESSION" | cut -d= -f2)
        [ -z $LEADER ] && exit
        OLDEST=$(pgrep -o -P $LEADER)
        [ -z $OLDEST ] && exit
        export $(grep -z DBUS_SESSION_BUS_ADDRESS /proc/$OLDEST/environ)
        [ -z "$DBUS_SESSION_BUS_ADDRESS" ] && exit
    fi
fi
notify-send "$@"
```

## 参考

<https://forums.linuxmint.com/viewtopic.php?t=279095>