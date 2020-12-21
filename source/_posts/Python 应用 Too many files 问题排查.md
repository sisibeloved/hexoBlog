---
title: Python 应用 Too many files 问题排查
date: 2020-11-20 09:43:00
categories: Backend
tags:
- Python
- Gunicorn
- Supervisor
- Linux
---

##  现象

上午发现调用算法接口报错，查看日志文件发现报错 `OSError: [Errno 24] Too many open files`

<!-- more -->

##  问题排查

###  1. 修改系统文件描述符限制

`ulimit -a` 查看系统限制，`ulimit -n 65535`修改只对当前终端生效。

在 `/etc/security/limits.d/20-nproc.conf` 中添加
```
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```

其中 `nofile` 是最大文件数量，`nproc` 是最大进程数量
然后执行 `sysctl -p`，问题仍存在。

###  2. 观察进程资源限制

`cat /proc/<PID>/limits`
其中有一行 `Max open files` 显示 Soft Limit 为 1024，Hard Limit 为4096，也就是说进程的最大文件数仍然被限制在了 4096。

###  3. 定位原因

因为是进程资源被限制，推测是 Supervisor 或者 Gunicorn 的问题，查找了下资料，发现果然是 Supervisor 的问题。因为使用 Supervisor 的进程管理机制，它会作为父进程 FORK 出子进程，鉴于父子关系，子进程允许打开的最大文件数不能超过父进程的阈值限制，但是 Supervisor 中 minfds 指令缺省设置的允许打开的最大文件数过小，进而导致子进程出现故障。

##  解决方案

修改 `/etc/supervisor/supervisor.conf`，将 `minfds`值修改为 10240，然后重启 Supervisor 主进程：`supervisorctl shutdown` 关闭，然后 `supervisord -c /etc/supervisor/supervisor.conf` 启动。

##  参考文档

[一次「Too many open files」故障](https://blog.huoding.com/2015/08/02/460)
[supervisor项目配置](https://www.cnblogs.com/brady-wang/p/11491026.html)