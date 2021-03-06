---
layout: post
title: 逗比小工具「fileMonitor、Aria2、ePing、rshell、mysql_monitor」
description: ""
keywords: ""
---

偶然间发现[doubi](https://github.com/ToyoDAdoubiBackup/doubi)项目好牛皮，虽然是一个逗比写的各种逗比脚本，但脚本真鸡儿好用~

效仿着也记录一下自己写的逗比小工具~

## 1、fileMonitor

基于Golang的文件监控小工具，使用[fsnotify](https://github.com/fsnotify/fsnotify)包，在**代码审计**或**CTF-AWD**中，无需依赖直接运行，使用十分方便。

代码地址：[https://github.com/SewellDinG/fileMonitor](https://github.com/SewellDinG/fileMonitor)

```
[Sewell]: ~/Documents/fileMonitor
➜  ./fileMonitor_mac -h
Usage of ./fileMonitor_mac:
  -path string
    	Input Your fileMonitorPath (default "./")

[Sewell]: ~/Documents/fileMonitor
➜  ./fileMonitor_mac ./
* 2020-01-17 21:36:08 Monitor Dir: /Users/sewellding/Documents/fileMonitor
  |-- 2020-01-17 21:36:54 Create: /Users/sewellding/Documents/fileMonitor/test
* 2020-01-17 21:36:54 Add Dir: /Users/sewellding/Documents/fileMonitor/test
  |-- 2020-01-17 21:37:02 Create: /Users/sewellding/Documents/fileMonitor/test/test.go
  |-- 2020-01-17 21:37:02 Create: /Users/sewellding/Documents/fileMonitor/test/test.go~
  |-- 2020-01-17 21:37:02 Write: /Users/sewellding/Documents/fileMonitor/test/test.go
  |-- 2020-01-17 21:37:02 Remove: /Users/sewellding/Documents/fileMonitor/test/test.go~
  |-- 2020-01-17 21:38:14 Remove: /Users/sewellding/Documents/fileMonitor/test/test.go
  |-- 2020-01-17 21:38:14 Remove: /Users/sewellding/Documents/fileMonitor/test
```

## 2、Aria2

下载境外的大型资源，往往由于自身带宽不够、代理速度慢等因素，导致资源下载速度无法忍受，🐢🐢🐢。

我的方法是：**① 利用Google GCP主机的大带宽优势，使用Aria2去下载资源；② 使用FileZilla的通用代理功能登陆VPS，利用SFTP拉回资源。**

[Aria2](https://github.com/aria2/aria2)是一款自由、跨平台命令行界面的下载管理器，支持的下载协议有：HTTP、HTTPS、FTP、SFTP、Bittorrent和Metalink，Aria2可以使用JSON-RPC和XML-RPC进行HTTP远程下载控制。

使用[doubi](https://github.com/ToyoDAdoubiBackup/doubi)项目的自动化编译脚本来安装Aria2：

```
sudo -s 获取root权限
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubiBackup/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
```

傻瓜式安装很方便，输出很清楚，也确实很逗比~

```
Aria2 一键安装管理脚本 [v1.1.10]
  -- Toyo | doub.io/shell-jc4 --
  0. 升级脚本
————————————
  1. 安装 Aria2
  2. 更新 Aria2
  3. 卸载 Aria2
————————————
  4. 启动 Aria2
  5. 停止 Aria2
  6. 重启 Aria2
————————————
  7. 修改 配置文件
  8. 查看 配置信息
  9. 查看 日志信息
 10. 配置 自动更新 BT-Tracker服务器
————————————
 当前状态: 已安装 但 未启动
 请输入数字 [0-10]: 4
 
 Aria2 简单配置信息：
 地址   : xxx.xxx.xxx.xxx
 端口   : 6800
 密码   : 38b754xxxxxxxxxfc41b0d4
 目录   : /usr/local/caddy/www/aria2/Download
```

我使用的Aria2前端是[AriaNg](https://github.com/mayswind/AriaNg)项目，仅有一个html文件。

将其下载到VPS上，使用Python快速起一个HTTP服务即可访问，随用随起，用完就停：`python -m http.server 8888`

![doubi_1](/assets/images/2020-01-29/doubi_1.png)

测试：下载 Kali2020.1 iso镜像

Google GCP下载速度可以达到70MB/s，大B，🐂！

FileZilla使用GCP的ss代理，拉回本地速度达到3+MB/s，也是大B，🐂！

本机单纯使用迅雷下载速度仅仅200Kb/s上下，Aria2下载速度不足百。

## 3、ePing

由于各个平台的ping命令不支持带协议的host，往往复制的url还需要单独提取host来使用。

ePing是基于Golang写的拓展ping工具，大致流程：**获取参数 --> 解析url --> 依据平台设置次数 --> 执行命令 --> 实时输出**。

代码地址：[https://github.com/SewellDinG/ePing](https://github.com/SewellDinG/ePing)

```
[Sewell]: ~/Documents/ePing ✗ master*
➜  ./eping https://baidu.com:443/urlpath/index.html
PING baidu.com (39.156.69.79): 56 data bytes
64 bytes from 39.156.69.79: icmp_seq=0 ttl=50 time=165.996 ms
64 bytes from 39.156.69.79: icmp_seq=1 ttl=50 time=79.287 ms
64 bytes from 39.156.69.79: icmp_seq=2 ttl=50 time=69.949 ms
64 bytes from 39.156.69.79: icmp_seq=3 ttl=50 time=54.314 ms

--- baidu.com ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 54.314/92.387/165.996/43.425 ms
```

将编译好的程序移动至环境变量中的程序中，或设置别名`alias ping='/path/eping'`来使用。建议使用前者，命名为eping。

## 4、rshell

输入IP与Port，自动生成Reverse Shell Cheatsheet。

代码地址：[https://github.com/SewellDinG/ReverseShellCheatsheet](https://github.com/SewellDinG/ReverseShellCheatsheet)

![rshell](https://github.com/SewellDinG/ReverseShellCheatsheet/raw/master/demo.png)

## 5、mysql_monitor

按提示要求输入MySQL数据库连接配置。

自动设置`SET GLOBAL general_log=on`和`SET GLOBAL log_output='table'`，即将SQL log写入mysql.general_log表，监听此表并实时输出。

代码地址：[https://github.com/SewellDinG/MySQLMonitor](https://github.com/SewellDinG/MySQLMonitor)

[![demo](https://github.com/SewellDinG/MySQLMonitor/raw/master/demo.png)](https://github.com/SewellDinG/MySQLMonitor/blob/master/demo.png)