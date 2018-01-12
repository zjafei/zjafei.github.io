---
title: shell学习笔记一
date: 2018-01-12 09:54:03
tags: ['shell','学习','package','包版本管理']
category: 'coding'
---
在计算机科学中，Shell俗称壳（用来区别于核），是指“提供使用者使用界面”的软件（命令解析器）。它类似于DOS下的command.com和后来的cmd.exe。它接收用户命令，然后调用相应的应用程序。
<!--more-->
## 给文件添加可以执行的权限
```bash
chmod +x shell-test.sh
```

## `#! /bin/bash` 执行解释器

## shell 执行内建命令，`PATH`变量列出的目录中寻找特定的命令

```bash
#! /bin/bash
ll;
```

终端输出 

```bash
./shell-test.sh: line 2: ll: command not found
```

如果使用

```bash
#! /bin/bash
ls --color=auto -alF;
```

终端正常输出

```bash
total 0
drwxrwxrwx 0 root root 4096 Jan 11 20:18 ./
drwxrwxrwx 0 root root 4096 Jan 11 20:16 ../
drwxrwxrwx 0 root root 4096 Jan 11 20:17 .git/
-rwxrwxrwx 1 root root  122 Jan 11 20:22 README.md*
-rwxrwxrwx 1 root root   59 Jan 11 20:26 shell-test.sh*
```

查看 `PATH` 变量的内容

```bash
echo $PATH;
```

## 使用 `&` 与 `;` 的区别

* 使用 `&`，不用等待 `&`前的代码结束就可以执行后面的代码：

```bash
#! /bin/bash
cd ../ & 
ls --color=auto -alF;
```

终端输出当前目录文件列表

* 使用 `;`，等待 `;`前的代码结束才可以执行后面的代码：

```bash
#! /bin/bash
cd ../ ; 
ls --color=auto -alF;
```
终端输出上一层目录文件列表

## 变量

* 变量名等号值要紧紧相贴
* 使用变量的时候变量名前面要加$

### 
```bash
#! /bin/bash
test='hello shell'; 
echo $test;
```

## echo 输出

### 加不加单引号都可以，但是对于一些特殊字符还是需要单引号的：
```bash
#! /bin/bash
echo hello world;
echo 'hello world';
echo 'hello world;'
```

## printf 的输出

```bash
#! /bin/bash
printf 'hello world \n';
printf 'hello world \n';
```
### printf 字符串的传参

```bash
#! /bin/bash
printf '%s %s \n' hello world;
```

今天就到这里，下次继续。