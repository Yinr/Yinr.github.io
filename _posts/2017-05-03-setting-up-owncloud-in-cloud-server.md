---
title: 在云服务器上搭建自己的 owncloud
date: 2017-05-03 21:12:04 +0800
category: cloud-server
tags: cloud-server owncloud LAMP
---

* content
{:toc}

最近看着我们 [Linux 中国](https://linux.cn) 的微信机器人 [LCbot](https://github.com/lctt/LCBot) 弄得很热闹呀，于是就也想试试。终于忍痛昨天在阿里云买了一个月的云服务器，学生价，9.9/月（优惠力度算是挺大了呢）。
（——*孩子你忘了五月份一波接一波的考试了么！！！* ——*...等我试一下就马上回去学习 T_T...*）

然而，服务器上准备好之后，准备新注册一个微信帐号，却发现现在微信注册必须要手机号。左右无果，就只好放一放暂时再想想别的用途了。

然后在今天看到了 owncloud（其实很早以前就见到过它，只是那时候没有服务器就没去弄），就决定弄个看看吧，反正云服务器闲着也是闲着，或许可以试着拿来给我们班弄个课件存储？

## 说明

其实本来我也并没有打算写这篇文章，但是因为我也是第一次弄服务器，第一次搭这种网络服务，弄了半天没弄好，然后就想着边弄边记录好了。

正是因为如此，我这篇的操作并不是严格按照官方文档的流程进行的，而是我自己一步一步的实践记录，我也不确定这样做出来之后的结果，所以本文仅供参考。

再说一下这台服务器的配置，这台服务器我选的是 CentOS 7.3，64 位，硬盘 40G，内存 2G。

下面正式开始吧。

## 安装软件包与依赖项

官方文档中有好种安装方法，我最初也在犹豫使用哪种方法。最后决定省事，直接根据官方提示，在 CentOS 的官方软件包仓库中下载。

首先添加仓库的信任：

    Run the following shell commands as root to trust the repository.

```
rpm --import https://download.owncloud.org/download/repositories/stable/CentOS_7/repodata/repomd.xml.key
```

然后添加 owncloud 官方仓库并安装 owncloud：

    Run the following shell commands as root to add the repository and install from there.

```
wget http://download.owncloud.org/download/repositories/stable/CentOS_7/ce:stable.repo -O /etc/yum.repos.d/ce:stable.repo
yum clean expire-cache
yum install owncloud
```

这样就会安装好 owncloud 和它所需要的依赖项 Apache、MySQL、PHP 等等。

但是，这里的安装其实就只是安装了软件包...麻烦还在后头

## 下一步...

等了好久终于安装好了之后，我发现我并不知道该干什么了。

现在所有东西都已经在我的服务器上了，然而却相当于什么都没有——我根本什么都不能用。

翻了好久官方文档。发现官方写的就是，安装完软件包就可以开 http://localhost/owncloud 来访问进行初始化配置了。然而...不知道是我没开 Apache 的原因还是它在这一步只能本地访问，反正我就是没办法继续了。

偷懒失败，那就只好一步步设置 Apache、PHP 什么的看看喽。参考教程：[在 RHEL/CentOS 7.0 中安装 LAMP](https://linux.cn/article-5789-1.html)

## 配置 Apache

现在的状况与全新安装 LAMP 比的好处是需要的东西都已经在服务器上了，但是这也会导致不能直接照着网上的教程走，那也只能这样走一步看一步喽。

有了 Apache，然后得启动 Apache 守护进程（再次说明，我这是在 CentOS 7.3 下的操作）：

```
systemctl start httpd
```

然后，教程提到得打开防火墙规则，但是却提示未运行 firewalld，那就手动运行起来看看好了：

```
systemctl start firewalld
firewall-cmd --add-service=http
```

随后，访问一下我们的服务器，应该就能出现 Apache 的测试页面了～不错～看样子暂时还没问题。

## 配置 PHP

之所以会有这一节，是因为按照流程一步一步来的话应该要出现这一部分，但是我试了下之后发现，并不需要特别配置，我们其实已经可以访问我们的 owncloud 页面了，地址在 http://localhost/owncloud。此外，在外网也是可以访问的，所以说明我之前无法访问的原因是在 Apache 服务没有打开。

到这里，其实已经可以开始你的 owncloud 之旅了。进入管理账户创建页面就可以直接创建和使用了，因为 owncloud 可以使用 SQLite 来作为后台数据库。但是如果要用 MySQL/MariaDB 的话或许还得再来多走几步。

既然走到这一步了，那就在继续试试看吧，就当连手也是可以的。

## 安装和配置 MariaDB

MariaDB 并没有包括在 owncloud 的依赖项中，所以要使用的话还的手动安装（看了下 MySQL 好像也没有）。

```
yum install mariadb-server mariadb
```

安装完之后就应该要启动它并且进行配置：

```
systemctl start mariadb
mysql_secure_installation
```

运行 `mysql_secure_installation` 之后，MairaDB 将会带你完成一些初始化和安全性配置，如设置 root 用户密码，禁止匿名用户，禁止 root 用户远程登录等等，设置完之后，在 owncloud 设置界面就可以用 MariaDB 的用户名和密码来开始使用 MariaDB 的 owncloud 啦～

## 自动启动设定

其实所有步骤都已经完成了，不过只是单次运行，当你重启服务器之后，他们都不会自动开始运行。

要让这些服务都自动运行，使用 `systemctl` 进行 enable 即可。哦对了，别忘了防火墙 firewall 的设置哦。

```
systemctl enable httpd
systemctl enable firewalld
firewall-cmd --permanent --add-service=http
systemctl enable mariadb
```

## 后记

其实，正如文章最初所说的，我是在尝试失败之后才开始写这篇文章然后记录操作过程的。所以起初我并没有想到事情会那么顺利，现在看来虽然没多少有分量的操作，但是却也让我对整个流程熟悉了不少吧。想来之后再搭建 LAMP 的话就会顺手很多。

我开始还有想用 Nginx 但是一时感觉太麻烦，就直接先顺着 LAMP 的教程做了，等之后有空再看看 Nginx 吧。

再次声明，这只是在我这边可以顺利完成，但是却不能保证其他环境下可以正常。我暂时也懒得找其他的资料，所以如果操作出现问题的话还请自行在去找找相关资源吧。

好了，收手收手，我得赶快去学习了 T_T...等之后稍微空闲了点再来看看还有什么可以完善的吧。
