---
title: shell学习笔记三
date: 2018-01-23 21:46:53
tags: ['shell','bash','脚本','终端','命令行']
category: 'coding'
---

Shell Script[1]  ，Shell脚本与Windows/Dos下的批处理相似，也就是用各类命令预先放入到一个文件中，方便一次性执行的一个程序文件，主要是方便管理员进行设置或者管理用的。但是它比Windows下的批处理更强大，比用其他编程程序编辑的程序效率更高，它使用了Linux/Unix下的命令。
<!--more-->

## 字符的操作
### grep 匹配字符
输入
```bash
echo 'string' | grep ^s
```

输出
```bash
string
```
### egrep 扩展 


### fgrep 快速 只是匹配文字
输入
```bash
echo 'string' | grep ^s
```

没有输出

### grep 都有什么用

只要是输出就可以匹配 so

* 高亮目录中的 root 

```bash
ll | grep root 
``` 

### sed 替换字符
```bash
# 删除某行      
sed '1d' textDir                                # 删除第一行 
sed '$d' textDir                                # 删除最后一行
sed '1,2d' textDir                              # 删除第一行到第二行
sed '2,$d' textDir                              # 删除第二行到最后一行
# 显示某行              
sed -n '1p' textDir                             # 显示第一行 
sed -n '$p' textDir                             # 显示最后一行
sed -n '1,2p' textDir                           # 显示第一行到第二行
sed -n '2,$p' textDir                           # 显示第二行到最后一行
# 使用模式进行查询              
sed -n '/ruby/p' textDir                        #查询包括关键字ruby所在所有行
sed -n '/\$/p' textDir                          #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义

# 增加一行或多行字符串                
sed '1a drink tea' textDir                      #第一行后增加字符串"drink tea"
sed '1,3a drink tea' textDir                    #第一行到第三行后增加字符串"drink tea"
sed '1a drink tea\nor coffee' textDir           #第一行后增加多行，使用换行符\n
# 代替一行或多行       
sed '1c Hi' textDir                             #第一行代替为Hi
sed '1,2c Hi' textDir                           #第一行到第二行代替为Hi
# 替换一行中的某部分 
# 格式：sed 's/要替换的字符串/新的字符串/g'   （要替换的字符串可以用正则表达式）
sed -n '/ruby/p' textDir | sed 's/ruby/bird/g'  #替换ruby为bird
# 插入
sed -i '$a bye' textDir                         #在文件textDir中最后一行直接输入"bye"
```

### cut 裁剪字符

#### 基本语法格式

```bash
cut [-bn] [file] 
cut [-c] [file] 
cut [-df] [file]
```

#### 主要参数

* -b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
* -c ：以字符为单位进行分割。
* -d ：自定义分隔符，默认为制表符。
* -f  ：与 -d 一起使用，指定显示哪个区域。

```bash
# 输出第一个字节
whoami | cut -b 1 
```

```bash
# 输出第一个字符
whoami | cut -c 1 
```

```bash
# 以 r 为分割符，取第二个元素
whoami | cut -d r -f 2 
```

### join 连接字符

把多个文本的指定字段合并到一行

```bash
# 1.txt
1 1
2 2
3 3
```

```bash
# 2.txt
1 a 
2 b
3 c
```

```bash
join 1.txt 2.txt
```
```bash
1 1 a 
2 2 b
3 3 c
```

```bash
join -o 2.2 1.1 1.txt  2.txt
```
```bash
a 1 
b 2
c 3
```

### awk 重新编排字段

```bash
history | awk '{print $1}' 
```

输出

```bahs
1
2
3
4
...
n
```
试试看

```bash
awk -F: '{ printf "User %s is really %s\n", $1, $5 }'  /etc/passwd
```

今天就到这里，下次继续。