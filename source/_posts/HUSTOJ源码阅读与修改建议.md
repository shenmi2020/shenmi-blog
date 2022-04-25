---
title: HUSTOJ源码阅读与修改建议
date: 2019-05-05 15:54
categories: [编程]
tags:
 - [hustoj]
---

从代码上HUSTOJ分为两大部分，core和web，分别对应判题和数据管理两大功能。

两者之间数据交换有两种方式：1、通过数据库，轮询。2、通过w3m实现的http请求。

两种方式的选择在判题端的配置文件/home/judge/etc/judge.conf中，HTTP_JUDGE=1则启用后者，默认为前者。

<!-- more -->

core分3部分，judged、judge_client、sim

其中judged为服务进程，d即daemon。负责轮询数据库或web端，提取判题队列。

当发现新任务时产生judge_client进程。

judge_client进程为实际判题程序，负责准备运行环境、数据，运行并监控目标程序的系统调用，采集运行指标，判断运行结果。

当配置为启用抄袭检查时，judge_client将调用sim，判断相似性结果，并写回数据库或web端。

sim为第三方应用程序，可进行语法分析判断文本相似度，通过检验的程序将由judge_client复制进题目数据的ac目录，成为新的参考样本。

w3m是linux下一个开源web文本浏览器，提供了基于命令行的http交互功能，这里用做http客户端，用curl-lib应该也可以实现，本人熟悉w3m不熟悉curl，因此偷懒。

 

web分两大部分，前端和admin目录下的管理程序。

前端无非是数据库的CRUD操作，关键功能是将用户提交的程序源码加入数据库的任务队列（solution表、souce_code表）。

管理程序提供具有administrator等高级权限的账号管理试题、账号等方面的功能。

其中FPS导入导出程序主要为XML格式的数据处理。

 

特别的，judged可以多重启动，通过增加基准目录参数指定启动位置（默认/home/judge），从而确定judge.conf的位置，并确定其他参数。

因此不但可以一个web服务器下挂多个判题服务器，也可以一台物理机器上同时启动任意多个相互独立的OJ系统。

 

实际使用中，使用开源的ispcp虚拟主机管理系统搭建多Web环境与hustoj协同工作取得了良好效果。

 

LiveCD的实现，通过uck工具解压出ubuntulivecd的chroot环境，并在其中删除oo、gnome等大型程序释放空间，然后用apt工具安装基础环境，安装配置lxde和hustoj。再使用uck重新打包形成iso。

 

升级方式：利用googlecode的svn服务，用svn客户端分别升级core和web,再编译core，并通过web提供可能的数据库升级。

livecd中的升级脚本为updagte-hustoj，可以用which命令查找其实际位置

------------------------------------------------------------

0、 准备知识

a)        最新系统源码可以用svn取得，或在下述地址直接浏览

i.  http://code.google.com/p/hustoj/source/browse/   墙外老版

ii.  https://github.com/zhblue/hustoj                         无墙新版

b)       系统分为Web和Core两个部分

c)        简化ER图

hustoj-db

d)       Web与core的连接方式有两种，实际运行可选其中一种

i.   数据库连接【默认】

1.        Web插入Solution表

2.        core轮询solution表，发现新纪录

3.        core更新solution表result等字段

4.        Web端轮询soltuion显示result等字段。

ii.   HTTP方式

1.        Web插入Solution表

2.        core访问Web端admin/problem_judge.php，发现新纪录

3.        core向Web端admin/problem_judge.php提交数据，problem_judge.php更新solution表result等字段

4.        Web端轮询soltuion显示result等字段。

1、 Web部分

a)        阅读配置文件，弄清各设置含义

i.      参考

 

db_info.inc.php


static $DB_HOST="localhost"; 数据库的服务器地址
static $DB_NAME="jol"; 数据库名
static $DB_USER="root"; 数据库用户名
static $DB_PASS="root"; 数据库密码
// connect db
static $OJ_NAME="HUSTOJ"; OJ的名字，将取代页面标题等位置HUSTOJ字样。
static $OJ_HOME="./"; OJ的首页地址
static $OJ_ADMIN="root@localhost"; 管理员email
static $OJ_DATA="/home/judge/data"; 测试数据所在目录，实际位置。
static $OJ_BBS="discuss";//"bbs" 论坛的形式，discuss为自带的简单论坛，bbs为外挂论坛，参考bbs.php代码。
static $OJ_ONLINE=false; 是否使用在线监控，需要消耗一定的内存和计算，因此如果并发大建议关闭
static $OJ_LANG="en"; 默认的语言，中文为cn
static $OJ_SIM=true; 是否显示相似度检测的结果。
static $OJ_DICT=true; 是否启用在线英字典
static $OJ_LANGMASK=1008; //1mC 2mCPP 4mPascal 8mJava 16mRuby 32mBash 1008 for security reason to mask all other language 用掩码表示的OJ接受的提交语言，可以被比赛设定覆盖。
static $OJ_EDITE_AREA=true;// 是否启用高亮语法显示的提交界面，可以在线编程，无须IDE。
static $OJ_AUTO_SHARE=false;//true: 自动分享代码，启用的话，做出一道题就可以在该题的Status中看其他人的答案。
static $OJ_CSS="hoj.css"; 默认的css,可以选择dark.css和gcode.css,具有有限的界面制定效果。
static $OJ_SAE=false; //是否是在新浪的云平台运行web部分
static $OJ_VCODE=true; 是否启用图形登录、注册验证码。
static $OJ_APPENDCODE=false; 是否启用自动添加代码，启用的话，提交时会参考$OJ_DATA对应目录里是否有append.c一类的文件，有的话会把其中代码附加到对应语言的答案之后，巧妙使用可以指定main函数而要求学生编写main部分调用的函数。
static $OJ_MEMCACHE=false;是否使用memcache作为页面缓存，如果不启用则用/cache目录
static $OJ_MEMSERVER="127.0.0.1"; memcached的服务器地址
static $OJ_MEMPORT=11211; memcached的端口
static $OJ_RANK_LOCK_PERCENT=0; //比赛封榜时间的比率，如5小时比赛设为0.2则最后1小时封榜。
static $OJ_SHOW_DIFF=false; //显示WrongAnswer时的对比

 

b)       制定自己的前台模板(即改变页面效果)

i.      复制template/bs3目录，放置在template目录中，并改为新模板名。

ii.     在db_info.inc.php中修改$OJ_TEMPLATE变量为新模板名

iii.   浏览前台，打开要修改的页面，根据地址栏修改新目录中对应的php、css、images等文件，保存后刷新页面看修改效果。

c)        模板制定成功以后应该有足够的知识开始修改template目录以外的部分了

d)       论坛

i.    建议集成GPL的phpbb，参考。

ii.    集成Discuz

1.   建议购买商业许可。

2.   参考/web/include/login-discuz.php

e)        比赛根据数据通过率排名，而不只看AC数量

i.             数据库solution表pass_rate字段表示改条通过率。

ii.             把contestrank.php中的solved字段变成浮点对待。

iii.            这里,修改积分方式，按照希望的方式积分。可能需要给TM增加字段$p_wa_best_rate记录每题最大通过率。

f)        对有志于重写整个前台的勇士

i.             希望你选择一种魔法师编程语言(node.js/ror/python/go)。

ii.             如果做不到前面那条，请做好长时间开发的心理准备。

iii.             理论上任何现存web编程模型都可以，推荐JSP/SSH(前方高能坑……)。

iv.             建议实现admin/problem_judge.php的仿真，方便直接集成原版core。(get/post/ servlet-mapping)

2、 Core部分

a)        阅读配置文件，弄清各设置含义

i.    参考


judge.conf  不要复制下面的注释进入实际文件，judged和judge_client不能识别#注释。

OJ_HOST_NAME=127.0.0.1 #mysql host ip
OJ_USER_NAME=root #mysql host username
OJ_PASSWORD=root #mysql host password
OJ_DB_NAME=jol #mysql DB name
OJ_PORT_NUMBER=3306 #mysql port
OJ_RUNNING=4 #max concurrent threads number of judge_client
OJ_SLEEP_TIME=5 #judged work interval
OJ_TOTAL=1 #Deprecated: total number of judged (hosts/processes)
OJ_MOD=0 #Deprecated: the number of this judged(host)
OJ_JAVA_TIME_BONUS=2 #java's extral time
OJ_JAVA_MEMORY_BONUS=512 #java's extral memory
OJ_SIM_ENABLE=0 #using sim
OJ_HTTP_JUDGE=0 #using http link to database(if enabled,mysql is not used anymore)
OJ_HTTP_BASEURL=http://127.0.0.1/JudgeOnline #http link basedir
OJ_HTTP_USERNAME=admin #account in db that has http_judge privilege
OJ_HTTP_PASSWORD=admin #password of this account
OJ_OI_MODE=0 #using oi (Olympiad in Informatics) mode
OJ_SHM_RUN=0 #using /dev/shm for fast running & low harddisk wear
OJ_USE_MAX_TIME=0 #use the max time of all testcase rather than total time
OJ_LANG_SET=0,1,2,3,4 #selective judge solution of languagesOJ_HOST_NAME=127.0.0.1 如果用mysql连接读取数据库，数据库的主机地址
OJ_USER_NAME=root 数据库帐号
OJ_PASSWORD=root 数据库密码
OJ_DB_NAME=jol 数据库名称
OJ_PORT_NUMBER=3306 数据库端口
OJ_RUNNING=4 judged会启动judge_client判题，这里规定最多同时运行几个judge_client
OJ_SLEEP_TIME=5 judged通过轮询数据库发现新任务，轮询间隔的休息时间，单位秒
OJ_TOTAL=1 老式并发处理中总的judged数量
OJ_MOD=0 老式并发处理中，本judged负责处理solution_id按照TOTAL取模后余数为几的任务。
OJ_JAVA_TIME_BONUS=2 Java等虚拟机语言获得的额外运行时间。
OJ_JAVA_MEMORY_BONUS=512 Java等虚拟机语言获得的额外内存。
OJ_SIM_ENABLE=0 是否使用sim进行代码相似度的检测
OJ_HTTP_JUDGE=0 是否使用HTTP方式连接数据库，如果启用，则前面的HOST_NAME等设置忽略。
OJ_HTTP_BASEURL=http://127.0.0.1/JudgeOnline 使用HTTP方式连接数据库的基础地址，就是OJ的首页地址。
OJ_HTTP_USERNAME=admin 使用HTTP方式所用的用户帐号（HTTP_JUDGE权限），该帐号登录时不能启用VCODE图形验证码，但可以登录成功后启用。
OJ_HTTP_PASSWORD=admin 密码
OJ_OI_MODE=0 是否启用OI模式，即无论是否出错都继续判剩余的数据，在ACM比赛中一旦出错就停止运行。
OJ_SHM_RUN=0 是否使用/dev/shm的共享内存虚拟磁盘来运行答案，如果启用能提高判题速度，但需要较多内存。
OJ_USE_MAX_TIME=1 是否使用所有测试数据中最大的运行时间作为最后运行时间，如果不启用则以所有测试数据的总时间作为超时判断依据。
OJ_LANG_SET=0,1,2,3,4 #判哪些语言的题目

OJ_COMPILE_CHROOT=0  是否在编译时使用chroot环境，避免某些编译期攻击。

OJ_TURBO_MODE=0 是否放弃用户表和问题表的数据一致性，以在大型比赛中添加更多的判题机来提高判题速度。

ii.      源码https://github.com/zhblue/hustoj/blob/master/trunk/core/judge_client/

b)       查阅Linux文档中关于下述关键词的内容

i.  Ptrace

ii.  Chroot

iii. Setuid

iv.  Proc

v. shm

c)        所有API限定在okcalls.h

d)       代码查重工具sim

i.   https://github.com/zhblue/hustoj/tree/master/trunk/core/sim

e)        对于计划改造Core来适应你自己的OJ前台的朋友

i. 参考1.c.iv

ii. 在judge_client.cc中搜索关键词wget

f)        HUSTOJ的沙箱模型

i.  相对openjudge.net的sandbox libraries而言并不严谨

ii.  对于OJ而言，基本满足需求

iii.  容易理解、容易实现、容易修改

f) 莫名其妙的Runtime Error，请点击RuntimeError打开详细信息，并做英译汉。

对于okcalls23/64.h进行修改，请只修改符合你的操作系统架构（32位/64位）的那个，0只能在首位，末尾必须为0。非0callid请加在中间任意位置。

CSDN网友的源码注释

http://blog.csdn.net/legan/article/details/40746829

http://blog.csdn.net/legan/article/details/40789939

 

 

 

result对应状态

0: "等待"

1: "等待重判"

2: "编译中"

3: "运行并评判"

4: "正确"

5: "格式错误"

6: "答案错误"

7: "时间超限"

8: "内存超限"

9: "输出超限"

10: "运行错误"

11: "编译错误"

12: "编译成功"

13: "运行完成"


文档
https://github.com/zhblue/hustoj/blob/master/wiki/hustoj%E6%96%87%E6%A1%A3%E5%A4%A7%E5%85%A8.pdf