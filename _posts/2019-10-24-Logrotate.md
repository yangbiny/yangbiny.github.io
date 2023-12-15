---
title:  "Logrotate"
date:   2019-10-24
desc: "Logrotate"
keywords: "Linux"
categories: [Linux]
tags: [linux]
icon: icon-html
---
# logrotate是什么
  logrotate是Linux中实现对日志进行切割的工具。当日志的大小达到一定的程度，或者是日志的时间达到一定的程度，需要要对日志进行归纳整理，logrotate可以很好的实现对日志的管理。

# 使用
  logrotate是由cron支持的定时执行任务，我们平时需要自定义对日志进行管理的时候，可以直接在/etc/logrotate.d中添加一个文件即可，执行时会自动执行该文件夹的所有文件。执行时间可以自定义（在cron中定义即可，如果自定义的话，也可以直接写脚本执行logrotate命令）

# 配置文件
```bash
/var/log/mongodb/*.log {     # 需要进行日志切割的日志文件的位置
	size=5M                  # 条件：大于这个条件时会进行切割
	daily                    # 执行周期：daily，weekly，monthly，yearly
    rotate 50                # 保留多少个，超过这个数时，最久的日志文件将会被删除
	copytruncate             # 把正在输出的日志拷贝一份出来，然后清空源文件
    dateext                  # 轮询的日志以日期结尾
	notifempty               # 忽略空文件
    missingok                # 如果文件不存在则忽略
	postrotate               # 脚本开始标志
	    /bin/kill -SIGUSR1 `cat /Impassive/logs/mongodb/mongo.pid 2>/dev/null` > /dev/null 2>&1
    endscript                # 脚本结束标志
    compress                 # 切割后压缩，也可以为nocompress
    delaycompress            # 切割时对上次的日志文件进行压缩
    create 640 nginx nginx   # 使用指定模式创建日志文件
}
还有其他的参数，欢迎补充
```

# 执行

## 测试
```bash
logrotate -d /etc/logrotate.d/test   # 只执行test文件，但是并不会真正的执行，可以用于测试
```
执行案例
![logrotate](http://impassive.blog.cone4.top/impassive-blog/logrotate-d-demo.png)

## 强制执行
```bash
logrotate -f /etc/logrotate.d/test   # 会真实的做日志轮询
```

# 注意
在mongo中，需要使用脚本
```bash
	postrotate               # 脚本开始标志
	    /bin/kill -SIGUSR1 `cat /Impassive/logs/mongodb/mongo.pid 2>/dev/null` > /dev/null 2>&1
    endscript                # 脚本结束标志
```
向mongo发送做日志切割的信号，同时在启动mongo的时候，需要配置参数logappend=true，然后logRotate才会生效（默认值是rename,可选reopen）。
当使用 -SIGUSR1 时，会触发mongo进行日志旋转。


如有错误，欢迎纠正。