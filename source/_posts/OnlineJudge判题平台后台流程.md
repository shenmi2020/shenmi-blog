---
title: OnlineJudge判题平台后台流程
date: 2019-06-04 16:48
categories: [编程]
tags:
 - [hustoj]

---

判题分两部分

1.其中judged为服务进程，d即daemon。负责轮询数据库，提取判题队列。当发现新任务时产生judge_client进程。
2.judge_client进程为实际判题程序，负责准备运行环境、数据，运行并监控目标程序的系统调用，采集运行指标，判断运行结果。

<!-- more -->

Judged流程
![判题流程](https://i.imgtg.com/2022/04/25/xdLlD.png)

初始化：

1.创建子进程pid_judged，并设置为会话的领头进程（umask(0),close(0~2)）
2.改变当前工作目录为“/home/judge”
3.将pid_judged写入文件“/home/judge/etc/judge.pid”,并且加入写锁（用来检查该服务进程是否已经在执行）
4.设置SIGQUIT,SIGKILL,SIGTERM来触发call_for_exit函数结束服务进程的执行

轮询：
5.连接数据库（host_name,user_name,password,db_name,port_number）
6.轮询数据库中的solution表，将未测评的用户提交扔进任务队列
7.从任务队列提取任务给“判题进程”（子进程），并标记该任务为“正在测评”
8.使用rlimit结构体和setrlimit()设置“判题进程”的允许的最大CPU运行时间，可以创建的最大文件字节数，可用内存最大字节数，可拥有最大进程数（200）
9.调用execl函数执行/usr/bin/judge_client,并传递参数为任务id,judge_client_id

Judge_client流程
![判题流程](https://i.imgtg.com/2022/04/25/xdGTF.png)

初始化：

1.设置工作目录为/home/judge

2.获取任务id,judge_client_id

3.连接数据库

4.设置工作目录为workdir=“/home/judge/run?”

5.解挂文件系统：workdir/proc,删除workdir/下的所有文件和目录

编译：


6.查询数据库获取题目id,用户id,语言类型

7.查询数据库获取 题目规定的最大运行时间time_lmt,最大内存mem_lmt

8.查询数据库获取用户提交的代码并写入Main.cc文件

9.创建子进程使用execvp函数执行g++命令编译Main.cc文件，并将编译信息写入ce.txt文件中

10.父进程调用waitpid函数来等待子进程编译结束，通过查看ce.txt文件来判断是否为编译错误，若为编译错误则退出

提取输入输出数据


11.在/home/judge/data/获取题目输入数据文件并保存路径为infile

12.将题目输入数据拷贝到workdir/data.in文件

13.获取题目输出数据文件并保存路径为outfile，创建user.out文件并保存路径为userfile

运行目标程序：


14.创建子进程pidApp，重定向输入输出流，从data.in读入数据，运行结果写入user.out，运行错误信息写入error.out

15.使用ptrace(PTRACE_TRACEME, 0, NULL, NULL)函数来使父进程跟踪自己

16.设置工作目录workdir为根目录（chroot()），提高系统安全性

17.使用rlimit结构体设置进程的CPU运行时间，可以创建最大文件字节数，可拥有最大进程数（1），拥有的堆栈数，可用内存数

18.alarm()设置定时器

19.调用execl使进程执行Main文件(目标程序)

监控目标程序：


20.父进程调用wait4(pidApp,&status,0,&ruse)来获取子进程暂停或中止时的返回状态和查看进程的使用资源情况

21.查看workdir/pro/pidApp/status里的VmPeak:属性的值，这个值是进程使用的最大内存数，如果超出限制，则ptrace(PTRACE_KILL,pidApp,NULL,NULL);退出监控


22.调用WIFEXITED(status)判断子进程是否正常结束

23.查看error.out文件，如果有内容则为运行错误，KILL掉子进程，监控结束

24.如果userfile的size > outfile的size*2，则为结果超出限制，kill掉子进程，监控结束

25.exitcode=WEXITSTATUS(status)获取子进程exit()返回的结束代码，如果为0或5则为正常，否则为异常结束，根据信号的类别给出相应错误，比如SIGALRM为计时器时间到了信号，SIGXCPU为运行时间到了信号，SIGXFSZ为输出文件大小超出信号等等


26.调用WIFSIGNALED(status)检查是否为异常结束（子进程通过信号结束）

27.调用sig=WTERMSIG(status)取得使子进程结束的信号编号，根据sig的类别给出相应的错误

28.调用ptrace(PTRACE_GETREGS,pidApp,NULL,&reg)来取得进程的寄存器信息(reg.REG_SYSCALL)，检查系统调用函数的使用情况，若为禁止的系统调用函数则KILL掉子进程并将运行错误写入结果AC_status，结束监控


29.调用ptrace(PTRACE_SYSCALL,pidApp,NULL,NULL)使子进程继续执行

30.在监控结束以后统计usedtime+=ruse.ru_utime(用户使用时间)+ruse.ru_stime(系统使用时间)

31.根据AC_status的值更新数据库


感谢hustoj开源代码及作者！
文章参考作者：cq_phqg
原文：https://blog.csdn.net/cq_phqg/article/details/46428987
