---
layout: post
title: Linux下安装ffmpeg
tags:
- ffmpeg
- linux
---

<!--*-->
<!--more-->

### 1. 下载[ffmpeg](http://ffmpeg.org/download.html )

```bash
wget https://ffmpeg.org/releases/ffmpeg-4.0.2.tar.bz2
tar -xjvf ffmpeg-4.0.2.tar.bz2
cd ffmpeg-4.0.2
```

### 2. 安装[yasm](http://yasm.tortall.net/Download.html )

yasm是一款汇编器，并且是完全重写了nasm的汇编环境，接收nasm和gas语法，支持x86和amd64指令集，所以这里安装一下yasm即可。

```bash
tar -xvzf yasm-1.3.0.tar.gz
cd yasm-1.3.0/
./configure
make
make install
```

### 3. 安装ffmpeg

```bash
#回到ffmpeg的文件夹
./configure --enable-shared --prefix=/app/ffmpeg
make
make install
```

  编译过程有点长，耐心等待完成之后执行 `cd /app/ffmpeg/` 进入安装目录，查看一下发现有bin,include,lib,share这4个目录，其中bin是ffmpeg主程序二进制目录，include是C/C++头文件目录，lib是编译好的库文件目录，share是文档目录。

### 4. 配置ffmpeg

* 将库文件导入linux

```bash
vi /etc/ld.so.conf.d/ffmpeg.conf
#加入下面这行
/app/ffmpeg/lib
#保存退出后，执行下面，使其生效
ldconfig
```

* 将路径添加进变量

```bash
vi /etc/profile
#到最后一行
export PATH=/app/ffmpeg/bin:$PATH
#保存退出后，执行下面，使其生效
source /etc/profile
```
### 5. 常用操作

* 合并mp4

  ```bash
  #转成ts后拼接
  ffmpeg -i 1.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 1.ts
  ffmpeg -i 2.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 2.ts
  ffmpeg -i "concat:1.ts|2.ts" -acodec copy -vcodec copy -absf aac_adtstoasc output.mp4
  ```

  