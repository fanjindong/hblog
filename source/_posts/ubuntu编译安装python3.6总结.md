---
title: ubuntu编译安装python3.6总结
---

如何在不影响系统自带python3.5的情况下，编译安装python3.6

## 1.安装依赖

依赖安装到不到位，会导致python安装后出现各种错误问题，所以这里附录了所有可能需要到依赖。

```bash
sudo apt-get install libssl1.0.0 libssl-dev tcl tk sqlite sqlite3 libbz2-1.0 libbz2-dev libexpat1 libexpat1-dev libgdbm3 libgdbm-dev  libreadline5 libreadline6 libreadline-dev libreadline6-dev libsqlite0 libsqlite0-dev libsqlite3-0 libsqlite3-dev openssl
```

## 2.下载与安装

下载地址：[python官网](https://www.python.org/)
使用`meke altinstall`.

*(若使用`make install` ,在系统中将会有两个不同版本到Python在/usr/bin/目录中，
这样会导致很多问题（图形桌面很多功能是基于python写的，你把自带python替换掉了，就会出现问题）)*

解压文件
`tar -xzvf Python-3.6.3.tgz -C  /tmp`
`cd  /tmp/Python-3.6.3/`

把Python3.6安装到 /usr/local 目录
`./configure --prefix=/usr/local`
`make -j8` *(-j8速度更快)*

`sudo make altinstall`

## 3.完成与使用

python3.6程序的执行文件：/usr/local/bin/python3.6

python3.6应用程序目录：/usr/local/lib/python3.6

pip3的执行文件：/usr/local/bin/pip3.6

pyenv3的执行文件：/usr/local/bin/pyenv-3.6

## 参考

[在 CentOS 7 上安装并配置 Python 3.6 环境](https://segmentfault.com/a/1190000009922582)