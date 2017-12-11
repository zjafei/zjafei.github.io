---
title: ubuntu16.0.4上python版本切换的问题
date: 2017-12-11 20:53:18
tags: ['ubuntu','python','版本','版本管理','切换']
category: 'coding'
---
事情的经过是这样的：
> 在安装一个npm项目的时候，终端输出了一行这样的错误

```bash
gyp verb check python checking for Python executable "python2" in the PATH
gyp verb `which` failed Error: not found: python2
```
<!--more-->
执行

```bash
which python
```

输出结果

```bash
/usr/bin/python
```

看来是版本的问题，
于是安装python2

```bash 
apt-get update     
apt-get install python2.7
```

终端还是输出了错误

```bash
gyp verb check python checking for Python executable "python2" in the PATH
gyp verb `which` failed Error: not found: python2
```

郁闷中，于是打开/usr/bin查看python

```bash
lrwxrwxrwx 1 root   root          18 Dec  9 17:39 python -> /usr/bin/python3.5*
```

原来python只是个软连接，执行命令

```bash
rm /usr/bin/python
ln -s /usr/bin/python2.7 /usr/bin/python 
```

修改软连接，python版本修改成功，npm包顺利安装成功！
