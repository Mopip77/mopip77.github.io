---
layout: post
title: 服务器部署selenium+chrome+chromedriver
tags:
- linux
- selenium
---

## 服务器部署selenium+chrome+chromedriver

### 系统版本

* Debian8 (jessie)

### 安装selenium

```bash
pip3 install selenium
```

### 安装chrome

在google-chrome官网(https://www.google.cn/chrome/)找到linux下载包（.deb）的链接。(可以先在自己电脑上下载，然后复制下载的链接)

使用wget下载, dpkg安装chorme。

```bash
wget url
dpkg -i xxxxx.deb
```

测试是否安装成功, 以及其版本：

```bash
whereis google-chrome
google-chrome -version	#查询Chrome版本
```

### 安装chromedriver

在网上查询完对应chrome版本的chromedriver后在http://chromedriver.storage.googleapis.com/index.html下载到服务器。（目前最新chrome67对应chromedriver2.40）

**注意，直接下载(解压)到/usr/bin/目录下，否则报错‘'chromedriver' executable needs to be in PATH’**

```bash
cd /usr/bin
wget URL
unzip chromedriver_linux64.zip
#没有unzip，用apt-get安装一下
rm chromedriver_linux64.zip
```

### 安装Xvfb

上述包安装完成后还不能直接运行webdriver，因为没有一个“屏幕”作为输出。

用Xvfb“虚构”一个屏幕。

```
apt-get install xvfb
#启动Xvfb
Xvfb -ac :7 -screen 0 1280x1024x8
export  DISPLAY=:7
```

每次重启服务器需要启动该服务，可将其写入启动脚本。

### 基于带界面的py脚本增加指令

```python
from selenium.webdriver.chrome.options import Options

chrome_option = Options()
chrome_option.add_argument('--headless')#无头模式，取代PhantomJS
chrome_option.add_argument('--no-sandbox')#禁止沙盒模式
chrome_option.add_argument('--disable-gpu')#禁止GPU加速
```



### 其他报错

* 运行python脚本时关于selenium的报错

> org.openqa.selenium.WebDriverException: unknown error: DevToolsActivePort file doesn't exist

意味着**ChromeDriver**无法启动/产生新的**WebBrowser，**即**Chrome浏览器**会话 。

添加**--disable-dev-shm-usage** 参数暂时解决问题。



* dpkg安装chrome时的报错

> google-chrome-stable depends on libasound2 (>= 1.0.16); however:
>  Package libasound2 is not installed.
> google-chrome-stable depends on libatk-bridge2.0-0 (>= 2.5.3); however:
>  Package libatk-bridge2.0-0 is not installed.
> google-chrome-stable depends on libgtk-3-0 (>= 3.9.10); however:
>  Package libgtk-3-0 is not installed.
> google-chrome-stable depends on libnspr4 (>= 2:4.9-2~); however:
>  Package libnspr4 is not installed.
> google-chrome-stable depends on libnss3 (>= 2:3.22); however:
>  Package libnss3 is not installed.
> google-chrome-stable depends on libx11-xcb1; however:
>  Package libx11-xcb1 is not installed.
> google-chrome-stable depends on libxss1; however:
>  Package libxss1 is not installed.
> google-chrome-stable depend
>dpkg: error processing package google-chrome-stable (--install):
> dependency problems - leaving unconfigured
>Processing triggers for man-db (2.7.0.2-5) ...
>Processing triggers for mime-support (3.58) ...
>Errors were encountered while processing:
> google-chrome-stable

```bash
apt-get update
apt-get install libgconf2-4 libnss3-1d libxss1
apt-get -f install
#然后再运行dpkg那条指令
```