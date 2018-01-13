---
title: shell学习笔记二
date: 2018-01-13 11:44:03
tags: ['shell','bash','脚本','终端','命令行']
category: 'coding'
---
`shell`是一种程序设计语言。作为命令语言，它交互式解释和执行用户输入的命令或者自动地解释和执行预先设定好的一连串的命令；作为程序设计语言，它定义了各种变量和参数，并提供了许多在高级语言中才具有的控制结构，包括循环和分支。
<!--more-->
## I/O 输入与输出

* a > b a输出给 b 
* a < b 接受d的输入
* a | b a 的输出作为 b 的输入

```bash
echo 'hello shell' > helloShell
```

输出 `hello shell` 到 `helloShell` 文件中

```text
hello eric
```

继续

```bash
echo 'hello eric' > helloShell
```

`helloShell` 文件中有 

```text
hello eric
```

继续

```bash
echo 'hello eric' >> helloShell
```

`helloShell` 文件中有 
```text
hello eric
hello eric
```

继续

```bash
tr 'l' 'x' < helloShell > hexxoShexx
```

会创建一个 `hexxoShexx` 文件，内容为：

```text
hexxo shexx
```


继续

```bash
echo 'hello shell ' | tr 'h' 't' > telloStell 
```

会创建一个 `telloStell` 文件，内容为：

```text
tello stell
```

## 小实验 tr 与 PATH 的结合使用 

```bash
echo $PATH | tr ':' '\n' > pathList.txt
```

创建 pathList.txt 文件，里面逐行列出了用户的环境变量

## grep 正则匹配输出

```bash
whoami | grep ^e
```

输出

```bash
eric
```

```bash
whoami | grep ^i
```

无输出

## 执行追踪 set -x

```bash
cat > get.sh
#! /bin/sh

set -x
echo 1st echo

set +x
echo 2nd echo
```

> `ctrl + d`

```bash
./get.sh
```

输出 

```bash
+ echo 1st echo
1st echo
+ set +x
2nd echo
```

今天就到这里，下次继续。