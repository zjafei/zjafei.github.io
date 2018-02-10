---
title: shell学习笔记四
date: 2018-02-10 21:43:03
tags: ['shell','bash','脚本','终端','命令行']
category: 'coding'
---
shell 命令重新初始化用户的登录会话。当给出该命令时，就会重新设置进程的控制终端的端口特征，并取消对端口的所有访问。然后 shell 命令为用户把进程凭证和环境重新设置为缺省值，并执行用户的初始程序。根据调用进程的登录用户标识建立所有的凭证和环境。
<!--more-->
## 文本处理工具
### sort 行排序

基本命令格式
```bash
sort [options] [file(s)] 
```

基本参数 
* -b   忽略每行前面开始出的空格字符。
* -c   检查文件是否已经按照顺序排序。
* -d   排序时，处理英文字母、数字及空格字符外，忽略其他的字符。
* -f   排序时，将小写字母视为大写字母。
* -i   排序时，除了040至176之间的ASCII字符外，忽略其他的字符。
* -m   将几个排序好的文件进行合并。
* -M   将前面3个字母依照月份的缩写进行排序。
* -n    依照数值的大小排序。
* -o<输出文件>   将排序后的结果存入指定的文件。
* -r   以相反的顺序来排序。
* -t<分隔字符>   指定排序时所用的栏位分隔字符
* -k, --key=POS1[,POS2]  指定关键字,可以指定多个关键字

输入
```bash
history |  sort -k2.1,3.1 # 从第二个字段的第一个字符比较到第三个字段的第一个字符
```

输出
```bash
  488  alert
   92  apt
   93  apt -update
   94  apt --update
   96  apt update
   98  apt
   95  aptupdate
  109  cat /etc/issue
  358  cat
  406  cat file
```

### uniq 行排序
作为管道修饰，用于排除重复的记录

基本参数 
* -c   相同值的计数
* -d   只显示重复的记录
* -u   只显示为重复的记录

输入
```bash
history | awk '{print $2,$3}'| sort -k1.1,2.1
```

输出
```bash
...
whoami |
whoami |
whoami |
```

输入
```bashechi
history | awk '{print $2,$3}'| sort -k1.1,2.1 | uniq
```

输出
```bash
...
who |
whoami
whoami |
```

### wc 计算行数，字数，字符数
wc命令的功能为统计指定文件中的字节数、字数、行数, 并将统计结果显示输出。

基本参数 
* -c   统计字节数。  
* -l   统计行数
* -w   统计字数

输入
```bash
curl www.baidu.com | wc
```

输出
```bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2381  100  2381    0     0  26727      0 --:--:-- --:--:-- --:--:-- 27056
      2     159    2381
```

输入
```bash
wc ./README.md
```

输出
```bash
 467  904 8158 ./README.md
```

### head 提取开头
用来显示档案的开头至标准输出中，默认head命令打印其相应文件的开头10行。 

基本参数 
* -q        隐藏文件名
* -v        显示文件名
* -c<字节>  显示字节数
* -n<行数>  显示的行数

输入
```bash
history | head -n 5
```

输出
```bash
  1  node -v
  2  git -v
  3  git status
  4  git branch
  5  cd workspace
```

输入
```bash
history | head -n -5 # 负数表示除去后面5行以为的其他内容
```

### tail 从指定点开始将文件写到标准输出
命令从指定点开始将文件写到标准输出.使用tail命令的-f选项可以方便的查阅正在改变的日志文件,tail -f filename会把filename里最尾部的内容显示在屏幕上,并且不但刷新,使你看到最新的文件内容. 

基本参数 
* -f 循环读取
* -q 不显示处理信息
* -v 显示详细的处理信息
* -c<数目> 显示的字节数
* -n<行数> 显示行数
* --pid=PID 与-f合用,表示在进程ID,PID死掉之后结束. 
* -q, --quiet, --silent 从不输出给出文件名的首部 
* -s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒

输入
```bash
history | tail -n 5
```

输出
```bash
  754  ll
  755  cd ~
  756  cd workspace/shell-test/
  757  ll
  758  history | tail
```

输入
```bash
history | tail -n +750 # 从750行开始显示到结尾
```

今天就到这里，下次继续。