---
title: centOS服务器添加服务添加启动项脚本
date: 2020-06-13 22:05:41
tags: ['centOS','service','启动项','rc.local','daemon','reload', 'systemctl']
category: 'coding'
---

再好的服务器都有重启的一天。重启后服务的自启动变的非常有必要。centOS的启动都写在`/etc/rc.local`这个文件里面。这个文件是个软连接，它指向的是`/etc/rc.d/rc.local `。默认这两个文件没有可执行的权限，这也是为什么很多启动项失败的原因<!--more-->

# 第一步 添加可执行权限
```bash
chmod +x /etc/rc.local 
chmod +x /etc/rc.d/rc.local 
```

# 第二步 启动服务查看服务状态
```bash
systemctl start rc-local.service 
systemctl status rc-local.service
```

# 第三步 制作一个服务通过systemctl启动

```bash
cd /etc/systemd/system
vim yourProjectName.service ## 可以把yourProjectName设置为你想起的服务名
```

yourProjectName.service的内容如下
```bash
[Unit]  
Description=yourProjectName #描述  
After=syslog.target network.target  #依赖  
 
[Service]  
Type=simple  
 
ExecStart=/usr/bin/java -jar /opt/javaapps/yourProjectName.jar  
#前面是java命令的绝对路径  后面是jar包的绝对路径  
ExecStop=/bin/kill -15 $MAINPID   ## kill -9 与 kill -15 -9 是立即停止 -15 等服务空闲后停止
 
User=root  
Group=root   
 
[Install]  
WantedBy=multi-user.target  
```

# 第四步 添加你要自启动的服务到文件里面
```bash
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local

systemctl start yourProjectName.service
```

如果启动服务遇到
```bash
Warning: nginx.service changed on disk. Run 'systemctl daemon-reload' to reload units.
```

解决方案
```bash
systemctl daemon-reload
```

>好了就写这么多,出去玩耍了 :)