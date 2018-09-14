---
title: GnuPG加密静态文件
date: 2018/7/15 14:12:01
tags:
- gnupg
- pgp
- 文件加密

---
gnupg 是一款开源的跨平台的加密软件，

> 要了解什么是GPG，就要先了解PGP。
1991年，程序员Phil Zimmermann为了避开政府监视，开发了加密软件PGP。这个软件非常好用，迅速流传开来，成了许多程序员的必备工具。但是，它是商业软件，不能自由使用。所以，自由软件基金会决定，开发一个PGP的替代品，取名为GnuPG。这就是GPG的由来。
http://www.ruanyifeng.com/blog/2013/07/gpg.html
GPG有许多用途，本文主要介绍静态文件加密。     

本文的使用环境为Linux命令行。如果掌握了命令行，Windows 或 Mac OS 客户端，就非常容易掌握。GPG并不难学，学会了它，从此就能轻松传递加密信息。建议读者一步步跟着教程做，对每条命令都自行测试。

### 下载gnupg
```
https://sourceforge.net/p/gpgosx/docu/Download/
```

### 使用密码来加密文件

```
gpg -c test.m4a
```
命令执行后，会需要输入密码，然后需要等待一会，文件越大耗时越久, 之后生成了一个加密文件test.m4a.pgp, 只需要备份test.m4a.pgp文件即可


### 使用密码来解密文件

```
gpg -d test.m4a.gpg > test.m4a
```
命令执行后，会需要输入密码，稍等片刻生成test.m4a，解密完成


