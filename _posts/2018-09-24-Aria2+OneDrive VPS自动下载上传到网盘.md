---
layout: post
title: Aria2+OneDrive VPS自动下载上传到网盘
tags:
- aria2
- linux
- onedrive
---

<!--*-->
<!--more-->

- 本教程基于Debian8

## 安装lnmp

```bash
apt-get update
#需要组件如下，可用apt-get install 安装
curl git unzip wget screen python(自带2.7就行)
#install lnmp
wget http://mirrors.linuxeye.com/lnmp-full.tar.gz
tar xzf lnmp-full.tar.gz
cd lnmp
./install.sh	#此过程很长需要15min左右
```

安装选项——仅列出所有需要**选y**的项，其余选n或默认

```bash
install Web server [y]
install Database   [y]
install PHP        [y]
install phpMyAdmin [y]
```

安装完毕后，提示是否重启VPS，选y重启。

## 绑定域名

需要两个域名。一个用于下载，一个用于操作网盘。

本教程使用两个二级域名dl2.mydomain 以及 pan2.mydomain，用A记录指向自己的服务器IP。

![Snipaste_2018-09-24_20-56-41.png](https://i.loli.net/2018/11/14/5bec24b0b8c37.png)

```bash
#进入lnmp文件夹
./vhost.sh
选 3. Use Let's Encrypt to Create SSL Certificate and Key
再输入上方的创建域名
其余全选n，再重复一次输入另一个域名
```

## 安装Aria2

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
#选1安装，安装完成后记录默认的随机密码，也可在脚本中修改密码
```

## 下载AriaNg和OneIndex源码

AriaNg是aira2的一个网页模板

```bash
#进入你所定义的下载域名文件夹
cd /data/wwwroot/dl2.mydomain
wget https://github.com/mayswind/AriaNg/releases/download/0.5.0/AriaNg-0.5.0.zip
unzip AriaNg-0.5.0.zip
rm AriaNg-0.5.0.zip
```

OneIndex是OneDrive的网页模板

```bash
#进入你所定义的网盘域名文件夹
cd /data/wwwroot/pan2.mydomain
git clone https://github.com/donwa/oneindex .
```

## 配置AriaNg和OneIndex

### AriaNg

用浏览器打开dl2.mydomain，此时显示未连接，选择左侧的**AriaNg设置**。

![Snipaste_2018-09-24_21-37-09.png](https://i.loli.net/2018/11/14/5bec24e1067bf.png)

选择RPC后，将刚刚记录的Aria2密码填入红色箭头框中，再点击弹出来的**重新加载页面**，状态变成已连接。

### Aria2

```bash
#进入Aria2的目录，默认在~/.aria2
#修改aria2.conf中的下载目录
dir=/data/aria2/Download
#并在配置文件末尾添加下面一行，即下载完成执行脚本
on-download-complete=/root/.aria2/upload2one.sh
```

编写 upload2one.sh

```bash
#!/bin/bash
path=$3
downloadpath='/data/aria2/Download'	#这里改成你定义的下载文件夹
if [ $2 -eq 0 ]
  then
    exit 0
fi
while true; do
filepath=$path
path=${path%/*};
if [ "$path" = "$downloadpath" ] && [ $2 -eq 1 ] #上传文件
    then
      /usr/local/php/bin/php /data/wwwroot/pan2.mydomain/one.php upload:file "$filepath" /uploadfolder/
    rm -rf "$filepath" #mydomain 改成自己的域名
    exit 0
elif [ "$path" = "$downloadpath" ] #上传文件夹
    then
      /usr/local/php/bin/php /data/wwwroot/pan2.mydomain/one.php upload:folder "$filepath"/ /uploadfolder/"${filepath##*/}"/ #mydomain 改成自己的域名
    rm -rf "$filepath"/
    exit 0
fi
done
```

给该脚本权限

```bash
chmod +x /root/.aria2/upload2one.sh
```

重启Aria2服务

```bash
/etc/init.d/aria2 restart
```

**Notice: **

两行pan2.mydomain改成自己网盘域名，后面跟着的uploadfolder是远程网盘中的根目录下的指定文件夹，可以改成自己喜欢的。

两行rm删除指令会在上传完成后删除下载的文件，在VPS容量有限时用。并且在传超过2G以上文件时上传可能失败，而删除指令仍会执行，建议注释掉。

### OneIndex

给www读写权限

```bash
chown -R www:www /data/wwwroot
```

浏览器打开pan2.mydomain

![Snipaste_2018-09-24_22-15-16.png](https://i.loli.net/2018/11/14/5bec24ff1928d.png)

这里需要OneDrive的授权，点击蓝色框登陆

![Snipaste_2018-09-24_22-18-07.png](https://i.loli.net/2018/11/14/5bec2511c8ff2.png)

将密钥复制到clinet sccret后，点击**知道了，返回到快速启动**

![Snipaste_2018-09-24_22-19-47.png](https://i.loli.net/2018/11/14/5bec252d13047.png)

将APP ID复制过去

## 提示
1. 默认的Aria2设置中，bt下载完成后会在**正在下载**队列里保种，即不会完成下载而且一直上传。如果使用限定流量的VPS建议如下操作。

![Snipaste_2018-09-24_22-35-36.png](https://i.loli.net/2018/11/14/5bec2543c8fb6.png)

最小分享率改成1.0。

如果实在不想保种可将最小做种时间改成0，即下完立刻完成。

## 两个上传脚本

**you-get下载后上传，具体规范看脚本**

```bash
#!/bin/bash

# 基于oneindex + you-get

# 新建opt文件夹 并且保持里面是空的
# 视频链接必须每行一条  保存在当前路径v.txt下
# 由于下完即删不保证上传，所以v.txt的内容不动，根据实际情况移除以下载的链接

opt="/root/videos"
video=( $(cat v.txt) )
count=0
# 如果有重名的下载文件，用count计数并添加在文件名前，以防重名， same_name 0,1表示是否会重名
same_name=0

# 删除没用的文件，字幕文件，弹幕文件
del_useless_file() {
	rm ${opt}/*.srt
	rm ${opt}/*.xml
}

for i in ${video[@]}
do
  let "count=$count+1"
  # 默认用137 即 1080p MP4 下载，下不了则用默认下载
  you-get --itag=137 -o ${opt} ${i}
  del_useless_file
  fp=$(ls $opt)
  if [ ! $fp ];then
  # 由于采用最简单的ls 来获取下载的视频，必须保证文件夹内只有一个文件
  # youtube 1080p 视频音频分开 必须安装ffmpeg才能自动合成
  you-get -o $opt ${i}
  del_useless_file
  fp=$(ls $opt)
  fi
  if [ $same_name -eq 1 ]
  then
    # mydomain 改成自己的域名
    php /data/wwwroot/pan2.mydomain/one.php upload:file "$opt/$fp" "/upload/${count}${fp}"
  else
    # mydomain 改成自己的域名
    php /data/wwwroot/pan2.mydomain/one.php upload:file "$opt/$fp" "/upload/${fp}"
  fi
  rm "$opt/$fp"
  echo "[*]\n第$count 个视频完成操作（不保证上传完成）\n"
done
```
这里如果使用screen来下载并检查是否下载成功，可能会无法用滚轮翻屏。`vi ~/.screenrc` ，添加`termcapinfo xterm* ti@:te@`，这样在screen中就可用滚轮翻屏。

**用`oneup <filename>`命令上传脚本**

```bash
#!/bin/bash
oneup() {
uploadFile=$1
if [ -d "$uploadFile" ]
# mydomain 改成自己的域名
then php /data/wwwroot/pan2.mydomain/one.php upload:folder "$uploadFile" /upload/
elif [ -f "$uploadFile" ]
# mydomain 改成自己的域名
then php /data/wwwroot/pan2.mydomain/one.php upload:file "$uploadFile" /upload/
fi
}
```