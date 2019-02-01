---
layout: post
title: Aria2+OneDrive VPS自动下载上传到网盘 简易版
tags:
- aria2
- linux
- onedrive
- rclone
---

- 由于之前的教程需要安装lnmp需要花费大量时间，所以找到了一种更为简易的方式
<!-- more -->
- 但是本方法需要本地和远端配合，所以需要本地有一个linux环境，windows下可以用linux子系统

## 安装配置aria2
- 在**服务器**都安装aira
```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
#选1安装，安装完成后记录默认的随机密码，也可在脚本中修改密码
```
- 修改**服务器**aria配置
```bash
# 新建下载文件夹
mkdir -p /data/Download
#进入Aria2的目录，默认在~/.aria2
#修改aria2.conf中的下载目录
dir=/data/aria2/Download
#并在配置文件末尾添加下面一行，即下载完成执行脚本
on-download-complete=/root/.aria2/upload2one.sh
```
修改完了得重启一下aria服务使其生效`/etc/init.d/aria2 restart`


## 安装AriaNg
> 由于是简易版安装，所以不在服务器上配置域名、安装AriaNg，直接通过本地AriaNg程序访问

在 https://github.com/mayswind/AriaNg/releases 中找到最新的AllInOne包下载
解压后是一个html文件，直接用浏览器打开即可


用浏览器打开后，此时显示未连接，选择左侧的**AriaNg设置**。

![Snipaste_2018-09-24_21-37-09.png](https://i.loli.net/2018/11/14/5bec24e1067bf.png)

选择RPC后，将刚刚记录的Aria2密码填入红色箭头框中
并且将RPC地址改成**你的服务器IP**
再点击弹出来的**重新加载页面**，状态变成已连接。

## 安装rclone
- rclone是一个命令行的网盘同步工具
  
在**服务器和本地**都安装
```bash
wget https://rclone.org/install.sh
bash install.sh
```
将onedrive信息加入到rclone
```bash
# 进入rclone配置
➜ rclone config
2018/11/30 21:53:03 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> one  # 随意起一个名字
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / A stackable unification remote, which can appear to merge the contents of several remotes
   \ "union"
 2 / Alias for a existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Dropbox
   \ "dropbox"
 9 / Encrypt/Decrypt a remote
   \ "crypt"
10 / FTP Connection
   \ "ftp"
11 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
12 / Google Drive
   \ "drive"
13 / Hubic
   \ "hubic"
14 / JottaCloud
   \ "jottacloud"
15 / Local Disk
   \ "local"
16 / Mega
   \ "mega"
17 / Microsoft Azure Blob Storage
   \ "azureblob"
18 / Microsoft OneDrive
   \ "onedrive"
19 / OpenDrive
   \ "opendrive"
20 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
21 / Pcloud
   \ "pcloud"
22 / QingCloud Object Storage
   \ "qingstor"
23 / SSH/SFTP Connection
   \ "sftp"
24 / Webdav
   \ "webdav"
25 / Yandex Disk
   \ "yandex"
26 / http Connection
   \ "http"
Storage> 18  # 选择Onedrive，编号可能不同
** See help for onedrive backend at: https://rclone.org/onedrive/ **

Microsoft App Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>   # 留空
Microsoft App Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>   # 留空
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n
For this to work, you will need rclone available on a machine that has a web browser available.
Execute the following on your machine:
	rclone authorize "onedrive"
Then paste the result below:
result>    
```

在这一步需要授权OneDrive，需要在本机上执行`rclone authorize "onedrive"`，在 http://127.0.0.1:53682/auth 上登录你的OneDrive
登录成功后会返回类似这样的信息
![ScreenShot_2018-11-30_21:54:43.png](https://i.loli.net/2018/11/30/5c01413994471.png)
将返回的内容和大括号一起复制到服务器

```bash
# 服务器后续
Choose a number from below, or type in an existing value
 1 / OneDrive Personal or Business
   \ "onedrive"
 2 / Root Sharepoint site
   \ "sharepoint"
 3 / Type in driveID
   \ "driveid"
 4 / Type in SiteID
   \ "siteid"
 5 / Search a Sharepoint site
   \ "search"
Your choice> 1
Found 1 drives, please select the one you want to use:
0:  (personal) id=6eb85ea240738233
Chose drive to use:> 0
Found drive 'root' of type 'personal', URL: https://onedrive.live.com/?cid=6eb85ea240738233
Is that okay?
y) Yes
n) No
y/n> y  
--------------------
[one]
type = onedrive
token = {"access_token":}
drive_id = 6eb85ea240738233
drive_type = personal
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
one                  onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q

```

## 编写上传脚本
`vi ~/.aria2/upload2one.sh`,将如下内容添加到该脚本中，并且将downloadpath，name，folder字段替换成你的信息

```bash
#!/bin/bash

filepath=$3	 #取文件原始路径，如果是单文件则为/Download/a.mp4，如果是文件夹则该值为文件夹内第一个文件比如/Download/a/1.mp4
path=${3%/*}	 #取文件根路径，如把/Download/a/1.mp4变成/Download/a
downloadpath='/data/Download'	#Aria2下载目录
name='one' #配置Rclone时的name
folder='/share'	 #网盘里的文件夹，如果是根目录直接留空
MinSize='10k'	 #限制最低上传大小，默认10k，BT下载时可防止上传其他无用文件。会删除文件，谨慎设置。
MaxSize='15G'	 #限制最高文件大小，默认15G，OneDrive上传限制。

if [ $2 -eq 0 ]; then exit 0; fi

while true; do
if [ "$path" = "$downloadpath" ] && [ $2 -eq 1 ]	#如果下载的是单个文件
    then
    rclone move -v "$filepath" ${name}:${folder} --min-size $MinSize --max-size $MaxSize
    rm -vf "$filepath".aria2	#删除残留的.aria.2文件
    rm "$filepath"
    exit 0
elif [ "$path" != "$downloadpath" ]	#如果下载的是文件夹
    then
    while [[ "`ls -A "$path/"`" != "" ]]; do
    rclone move -v "$path" ${name}:/${folder}/"${path##*/}" --min-size $MinSize --max-size $MaxSize --delete-empty-src-dirs
    rclone delete -v "$path" --max-size $MinSize	#删除多余的文件
    rclone rmdirs -v "$downloadpath" --leave-root	#删除空目录，--delete-empty-src-dirs 参数已实现，加上无所谓。
    done
    rm -vf "$path".aria2	#删除残留的.aria2文件
    rm -rf "$path"
    exit 0
fi
done
```

最后给脚本权限`chmod +x ~/.aria2/upload2one.sh`

最后可以下载一个东西试试了～

- 由于上传方式不同所以比之前的方法会慢一点，如果在意上传的速度，还是使用之前的方法吧