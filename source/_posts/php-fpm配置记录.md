---
title: php-fpm配置记录
date: 2023-10-25 16:48:07
tags:
 - [php]
---

Socket是使用unix domain socket连接套接字/dev/shm/PHP-cgi.sock（很多教程使用路径/tmp，而路径/dev/shm是个tmpfs，速度比磁盘快得多）,在服务器压力不大的情况下，tcp和socket差别不大，但在压力比较满的时候，用套接字方式，效果确实比较好。

<!-- more -->

```
[www] #进程池的名字；
user  = www
group = www #以什么用户什么组的权限来运行[www]这个进程池,用户，必须要设置,用户组，如果没有设置，则默认用户的组被使用;
listen.backlog = 65535 #子进程未accept处理的客户端请求队列大小，-1 on FreeBSD and OpenBSD，其他平台默认65535，高并发时重要，合理设置会及时处理排队的请求；太大会积压太多，处理完后nginx在前面都等超时断开这个和fpm的socket连接了，就杯具了。不要用-1，建议1024以上，最好是2的幂值（注意：不同版本的默认值是不同的，php5多是128，php7多是511）。
#1：一个池共用一个backlog队列，所有的池进程都去这个队列里accept连接； 
#2：最大数量受限于系统配置cat /proc/sys/net/core/somaxconn，系统配置修改：vim /etc/sysctl.conf，增加net.core.somaxconn =1024则最大为1024，然后php最大的backlog可以到1024。
listen = 127.0.0.1:9000 #进程池[www]的监听端口,可用格式为:'ip:port'，'port'，'/path/to/unix/socket'。每个进程池都需要设置。如果nginx和php在不同的机器上，只能用机器真实ip+端口的格式，如果在同一台机器上，还可以选择unix soxket方式，这种理论上速度会提升31%，但高并发下不够稳定。
listen.owner = www
listen.group = www
listen.mode = 0666 #unix socket设置选项，如果使用tcp方式访问，这里注释即可。
listen.acl_users = www,php #当系统支持 POSIX ACL（Access Control Lists）时，可以设置使用此选项。 当设置了的时候，将会忽略 listen.owner 和 listen.group。值是逗号分割的用户名列表。 PHP 5.6.5 起可用。
listen.acl_groups=web #参见 listen.acl_users。 值是逗号分割的用户组名称列表。 PHP 5.6.5 起可用。
listen.allowed_clients = 127.0.0.1 #允许访问FastCGI进程的IP白名单，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。每个地址是用逗号分隔。如果没有设置或者为空，则允许任何服务器请求连接。
process.priority = -19 #该池进程的权限，同样要master进程是root用户才有效，和全局那个一样，不设置的话会继承master进程的优先级。
pm = dynamic #可选static,dynamic,ondemand,也就是说php-fpm有三种进程管理模式，默认dynamic。
#1：static,固定启动若干（即pm.max_children）php进程，保持不变。
#有效配置：pm.max_children
#2：ondemand，这种模式刚启动时不会启动任何php进程，只有php-fpm接收到请求时才会根据需求启动php进程，最大为pm.max_children个，另外若php进程空闲时间达到pm.process_idle_timeout（单位s）,就会kill掉该进程。
#有效配置：pm.max_children，pm.process_idle_timeout
#3：dynamic，这种是最常用的，根据相关配置动态调整php进程个数;
#有效配置如下：
#pm.max_children ： 最大php进程数；
#pm.min_spare_servers：最小的空闲php进程数,少与该值会启动php进程（这里的空闲并不是指完全空闲的php进程，可以直接理解为启动的php进程就好了，把空闲二字去掉更贴切）；
#pm.max_spare_servers：最大的空闲php进程数，多余的会被kill；
#pm.start_servers ： php-fpm启动时的php进程数，它的值需要在min_spare_servers和max_spare_servers之间，默认值：min_spare_servers(max_spare_servers - min_spare_servers) / 2；
#pm.max_requests ，默认为0（此时等于PHP_FCGI_MAX_REQUESTS）。为了便于描述，此处假设它的值是500，那么这个参数的作用是一个php进程(即fpm的一个子进程)处理500个请求后会被kill,然后再启动一个php进程，这样可以防止因为内存泄漏导致的php进程占用内存过高的问题。
access.log = var/log/php-fpm/$pool-access.log #访问文件日志;
access.format = "%R - %u %t “%m %r%Q%q” %s %f %{mili}d %{kilo}M %C%%" #设定访问日志的格式。
slowlog = /var/log/php-fpm/$pool-slow.log #慢请求日志；
request_slowlog_timeout   #默认为0（不启用），此处假设它的值是10，则超过10s未响应的请求就是慢请求，会被记录到慢请求日志中；
request_terminate_timeout #默认为0（不启用），此处假设它的值是20，则若某个请求超过20s未响应，相应的php进程会被kill掉，和php.ini中的max_execution_time效果类似。
php_value ,php_flag, php_admin_value , php_admin_flag #设置php.ini中的配置，后二者相比前两者，不能被 PHP 代码中的 ini_set() 及相似函数覆盖。

#最重要的就是pm相关的几个配置了，还有一些配置采用默认就好，详情见官网。

【全局配置】
pid = run/php-fpm.pid #pid设置。
error_log = log/php-fpm.log #错误日志。
log_level = notice #错误级别。上面的php-fpm.log纪录的错误等级。可用级别为：alert（必须立即处理），error（错误情况），warning（警告情况），notice（一般重要信息），debug（调试信息）。默认：notice。
syslog.facility = daemon #把日志写进系统log，linux还不够熟悉，暂时不用理会。
syslog.ident = php-fpm #系统日志标示（前缀），如果跑了多个fpm进程池，需要用这个来区分日志是谁的。
emergency_restart_threshold = 5
emergency_restart_interval = 60 #表示在60s内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值。0 表示‘关闭该功能’。默认值: 0 (关闭)。
process_control_timeout = 0 #设置子进程接受主进程复用信号的超时时间。可用单位：s(秒)，m(分)，h(小时)，或者 d(天) 默认单位: s(秒)。默认值: 0。
process.max = 128 #当动态管理子进程时，fpm最多能fork多少个进程，默认0表示无限制，这是所有进程池能启动子进程的总和，谨慎使用。
process.priority = -19 #设置子进程的优先级，在master进程以root用户启动时有效；如果没有设置，子进程会继承master进程的优先级，值范围-19(最高)到20(最低)，默认不设置。
rlimit_files = 1024 #设置master进程最多能打开的文件，默认为系统的值。
rlimit_core = 0 #master进程核心rlimit限制值；可选unlimited或>=0的整数，默认为系统的值。
events.mechanism = epoll #事件处理机制，默认自动检测，可选值：select（any POSIX os）， poll(any POSIX os)， epoll(linux>=2.5.44)， kqueue（FreeBSD >= 4.1,OpenBSD >= 2.9, NetBSD >= 2.0）， /dev/poll（Solaris >= 7），port（Solaris >= 10）。linux>=2.5.44会默认epoll,效果最好的IO方式。
systemd_interval = 10s #当fpm被设置为系统服务时，多久向服务器报告一次状态，单位有s,m,h。
daemonize = yes #作为守护进程运行php-fpm。默认值为yes。
```