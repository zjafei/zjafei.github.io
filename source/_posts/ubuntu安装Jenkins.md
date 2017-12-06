---
title: ubuntu安装Jenkins
date: 2017-12-06 19:40:19
tags: ['CI','持续集成','jenkins','continuous integration','ubuntu']
category: 'coding'
---

## A. 首先查看是否安装了`JDK`和`JRE`（建议`openjdk-7-jre`和`openjdk-7-jdk`）：

```bash
java -version
```
如果没有安装，执行：

1.更新包指数

```bash
sudo apt-get update
```

2.安装Java运行时环境

```bash
sudo apt-get install default-jre
```

3.安装JDK

```bash
sudo apt-get install default-jdk
```
<!--more-->
## B. 下面的步骤是正式安装`Jenkins`：
1.获取秘钥
```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```

2.添加源列表
```bash
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

3.更新
```bash
sudo apt-get update
```

4.安装
```bash
sudo apt-get install jenkins
```

Jenkins服务默认运行在8080端口，如果Jenkins服务不能启动，可以编辑：
```bash
/etc/default/jenkins
```
替换
```txt
HTTP_PORT=8080
```
为
```txt
HTTP_PORT=8000
```
8000为你所希望Jenkins服务的端口

5.打开浏览器输了你的主机ip和Jenkins的服务端口如：192.168.1.22:8080

6.页面会索要一个key，key的地址 `/var/lib/jenkins/secrets/initialAdminPassword`