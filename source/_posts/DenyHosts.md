---
title: 关于使用DenyHosts阻止SSH暴力破解密码
date: 2018-03-21 20:11:49
categories:
- keep learning
tags:
- Linux
- Python
- VPS
- 安全

---

## 查看用户登录历史
```bash
    /var/run/utmp（用于记录当前打开的会话）被who和w工具用来记录当前有谁登录以及他们正在做什么，
		而uptime用来记录系统启动时间。
    /var/log/wtmp （用于存储系统连接历史记录）被last工具用来记录最后登录的用户的列表。
    /var/log/btmp（记录失败的登录尝试）被lastb工具用来记录最后失败的登录尝试的列表。
```
    
> utmpdump，这个小程序来自sysvinit-tools包，可以用于转储二进制日志文件到文本格式的文件以便检查。此工具默认在CentOS 6和7系列上可用。utmpdump收集到的信息比先前提到过的工具的输出要更全面，这让它成为一个胜任该工作的很不错的工具。除此之外，utmpdump可以用于修改utmp或wtmp。如果你想要修复二进制日志中的任何损坏条目，它会很有用。


 显示失败的登录尝试
```bash
    utmpdump /var/log/btmp
```

## 安装DenyHosts
>DenyHosts是Python语言写的一个程序，它会分析sshd的日志文件（/var/log/secure），当发现重 复的攻击时就会记录IP到/etc/hosts.deny文件，从而达到自动屏IP的功能。
>用DenyHosts可以阻止试图猜测SSH登录口令，它会分析/var/log/secure等日志文件，当发现同一IP在进行多次SSH密码尝试时就会记录IP到/etc/hosts.deny文件，从而达到自动屏蔽该IP的目的。

```bash
	# Debian/Ubuntu： 
	$ sudo apt-get install denyhosts 
	# RedHat/CentOS 
	$ yum install denyhosts 
	# Archlinux 
	$ yaourt denyhosts 
	# Gentoo 
	$ emerge -av denyhosts 
```

<!--more-->
启动

```bash
	service denyhosts start
```
默认配置 /etc/denyhosts.conf：

```bash
$ vi /etc/denyhosts.conf 
	SECURE_LOG = /var/log/auth.log #ssh 日志文件，它是根据这个文件来判断的。 
	HOSTS_DENY = /etc/hosts.deny #控制用户登陆的文件 
	PURGE_DENY = #过多久后清除已经禁止的，空表示永远不解禁 
	BLOCK_SERVICE = sshd #禁止的服务名，如还要添加其他服务，只需添加逗号跟上相应的服务即可 
	DENY_THRESHOLD_INVALID = 5 #允许无效用户失败的次数 
	DENY_THRESHOLD_VALID = 10 #允许普通用户登陆失败的次数 
	DENY_THRESHOLD_ROOT = 1 #允许root登陆失败的次数 
	DENY_THRESHOLD_RESTRICTED = 1 
	WORK_DIR = /var/lib/denyhosts #运行目录 
	SUSPICIOUS_LOGIN_REPORT_ALLOWED_HOSTS=YES 
	HOSTNAME_LOOKUP=YES #是否进行域名反解析 
	LOCK_FILE = /var/run/denyhosts.pid #程序的进程ID 
	ADMIN_EMAIL = root@localhost #管理员邮件地址,它会给管理员发邮件 
	SMTP_HOST = localhost 
	SMTP_PORT = 25 
	SMTP_FROM = DenyHosts <nobody@localhost> 
	SMTP_SUBJECT = DenyHosts Report 
	AGE_RESET_VALID=5d #用户的登录失败计数会在多久以后重置为0，(h表示小时，d表示天，m表示月，w表示周，y表示年) 
	AGE_RESET_ROOT=25d 
	AGE_RESET_RESTRICTED=25d 
	AGE_RESET_INVALID=10d 
	RESET_ON_SUCCESS = yes #如果一个ip登陆成功后，失败的登陆计数是否重置为0 
	DAEMON_LOG = /var/log/denyhosts #自己的日志文件 
	DAEMON_SLEEP = 30s #当以后台方式运行时，每读一次日志文件的时间间隔。 
	DAEMON_PURGE = 1h #当以后台方式运行时，清除机制在 HOSTS_DENY 中终止旧条目的时间间隔,这个会影响PURGE_DENY的间隔。 
```
