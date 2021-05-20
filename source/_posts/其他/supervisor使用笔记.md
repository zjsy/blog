---
title: supervisor使用笔记
comments: true
tags:
  - 进程管理
abbrlink: 13046
categories:
  - 其他
date: 2019-06-09 16:26:57
---

## 背景

在配置thinkphp5和laravel队列的时候往往要配置用到supervisor做队列的监听进程的管理,这里就补充一点关于supervisor的知识,本文大部分内容转自 [supervisor 管理进程简明教程](https://blog.csdn.net/woshixiaosimao/article/details/54315258),并在此基础上做了一些笔记

## 介绍

superviosr是一个Linux/Unix系统上的进程监控工具，他/她upervisor是一个Python开发的通用的进程管理程序，可以管理和监控Linux上面的进程，能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能自动重启。不过同daemontools一样，它不能监控daemon进程

### 优势

1. **简单**

   通常管理linux进程的时候，一般来说都需要自己编写一个能够实现进程`start/stop/restart/reload`功能的脚本，然后丢到/etc/init.d/下面。这么做有很多不好的地方，第一我们要编写这个脚本，这就很耗时耗力了。第二，当这个进程挂掉的时候，linux不会自动重启它的，想要自动重启的话，我们还要自己写一个监控重启脚本。而supervisor则可以完美的解决这些问题。好，怎么解决的呢，其实supervisor管理进程，就是通过 fork/exec的方式把这些被管理的进程，当作supervisor的子进程来启动。这样的话，我们只要在supervisor的配置文件中，把要管理的进程的可执行文件的路径写进去就OK了。这样就省下了我们如同linux管理进程的时候自己写控制脚本的麻烦了。第二，被管理进程作为supervisor的子进程，当子进程挂掉的时候，父进程可以准确获取子进程挂掉的信息的，所以当然也就可以对挂掉的子进程进行自动重启了，当然重启还是不重启，也要看你的配置文件里面有木有设置autostart=true了，这是后话。

2. **精确**

   为啥说精确呢？因为linux对进程状态的反馈，有时候不太准确。为啥不准确？这个楼主也不知道啊，官方文档是这么说的，知道的告诉楼主一下吧，感激不尽。而supervisor监控子进程，得到的子进程状态无疑是准确的。进程组supervisor可以对进程组统一管理，也就是说咱们可以把需要管理的进程写到一个组里面，然后我们把这个组作为一个对象进行管理，如启动，停止，重启等等操作。而linux系统则是没有这种功能的，我们想要停止 一个进程，只能一个一个的去停止，要么就自己写个脚本去批量停止。

3. **集中式管理**

    supervisor管理的进程，进程组信息，全部都写在一个ini格式的文件里就OK了。而且，我们管理supervisor的时候的可以在本地进行管理，也可以远程管理，而且supervisor提供了一个web界面，我们可以在web界面上监控,管理进程。 当然了，本地，远程和web管理的时候，需要调用supervisor的xml_rpc接口，这个也是后话。

4. **有效性**

   当supervisor的子进程挂掉的时候，操作系统会直接给supervisor发信号。而其他的一些类似supervisor的工具，则是通过进程的pid文件，来发送信号的，然后定期轮询来重启失败的进程。显然supervisor更加高效。。。至于是哪些类似supervisor工具，这个楼主就不太清楚了，楼主还听说过god,director，但是没用过。有兴趣的朋友可以玩玩  

5. **可扩展性**

   supervisor是个开源软件，牛逼点的，可以直接去改软件。不过咱们大多数人还是老老实实研究supervisot提供的接口吧，supervisor主要提供了两个可扩展的功能。一个是event机制，这个就是楼主这两天干的活要用到的东西。再一个是xml_rpc,supervisor的web管理端和远程调用的时候，就要用到它了。

6. **权限**

    大伙都知道linux的进程，特别是侦听在1024端口之下的进程，一般用户大多数情况下，是不能对其进行控制的。想要控制的话，必须要有root权限。而supervisor提供了一个功能，可以为supervisord或者每个子进程,设置一个非root的user，这个user就可以管理它对应的进程了。

7. **兼容性，稳定性**

### 组成部分

·         supervisord：服务守护进程

·         supervisorctl：命令行客户端

·         Web Server：提供与supervisorctl功能相当的WEB操作界面

·         XML-RPC Interface：XML-RPC接口

## 安装

推荐 python 第三方包的安装方法, *P.S.官网有更多方式*

```
pip install supervisor
```

## 使用说明

使用supervisor很简单，只需要修改一些配置文件，就可以使用了。

### 查看默认配置

[root@iZwz99i40ignvwjuinf024Z ~]# echo_supervisord_conf

即可看到默认配置情况，但是一般情况下，我们都不要去修改默认的配置，而是将默认配置重定向到另外的文件中，不同的进程运用不同的配置文件去对默认文件进行复写即可。

### 默认配置说明

>**默认的配置文件是下面这样的，但是这里有个坑需要注意，supervisord.pid 以及 supervisor.sock 是放在 /var/tmp 目录下，但是 /tmp 目录是存放临时文件，里面的文件是会被 Linux 系统删除的，一旦这些文件丢失，就无法再通过 supervisorctl 来执行 restart 和 stop 命令了，将只会得到 unix:///tmp/supervisor.sock 不存在的错误 。**

```ini
[unix_http_server]
;socket文件的路径，supervisorctl用XML_RPC和supervisord通信就是通过它进行的。如果不设置的话，supervisorctl也就不能用了
file=/var/tmp/supervisor.sock
;这个简单，就是修改上面的那个socket文件的权限为0700,不设置的话，默认为0700。 非必须设置
;chmod=0700
;这个一样，修改上面的那个socket文件的属组为user.group不设置的话，默认为启动supervisord进程的用户及属组。非必须设置
;chown=nobody:nogroup
;使用supervisorctl连接的时候，认证的用户不设置的话，默认为不需要用户。 非必须设置
;username=user
;和上面的用户名对应的密码，可以直接使用明码，也可以使用SHA加密,如：{SHA}82ab876d1387bfafe46cc1c8a2ef074eae50cb1d 默认不设置。。。非必须设置
;password=123456

; 侦听在TCP上的socket，Web Server和远程的supervisorctl都要用到他不设置的话，默认为不开启。非必须设置
[inet_http_server]
;这个是侦听的IP和端口，侦听所有IP用 :9001或*:9001。只要上面的[inet_http_server]开启了，就必须设置它
port=127.0.0.1:9001
;username=user              ; 这个和上面的uinx_http_server一个样。非必须设置
;password=123               ; 这个也一个样。非必须设置

;这个主要是定义supervisord这个服务端进程的一些参数的这个必须设置，不设置，supervisor就不用干活了
[supervisord]                
;这个是supervisord这个主进程的日志路径，注意和子进程的日志不搭嘎。 默认路径$CWD/supervisord.log，$CWD是当前目录。。非必须设置
logfile=/var/tmp/supervisord.log
;这个是上面那个日志文件的最大的大小，当超过50M的时候，会生成一个新的日志文件。当设置为0时，表示不限制文件大小 默认值是50M，非必须设置.     
logfile_maxbytes=50MB        
;日志文件保持的数量，上面的日志文件大于50M时，就会生成一个新文件。文件数量大于10时，最初的老文件被新文件覆盖，文件数量将保持为10当设置为0时，表示不限制文件的数量。默认情况下为10。。。非必须设置
logfile_backups=10
;日志级别，有critical, error, warn, info, debug, trace, or blather等默认为info。。。非必须设置项
loglevel=info
;supervisord的pid文件路径.默认为$CWD/supervisord.pid。。。非必须设置
pidfile=/var/tmp/supervisord.pid
;如果是true，supervisord进程将在前台运行默认为false，也就是后台以守护进程运行。。。非必须设置
nodaemon=false
;这个是最少系统空闲的文件描述符，低于这个值supervisor将不会启动。系统的文件描述符在这里设置cat /proc/sys/fs/file-max默认情况下为1024。。。非必须设置
minfds=1024
;最小可用的进程描述符，低于这个值supervisor也将不会正常启动。ulimit  -u这个命令，可以查看linux下面用户的最大进程数默认为200。。。非必须设置
minprocs=200
;进程创建文件的掩码默认为022。。非必须设置项
;umask=022
;这个参数可以设置一个非root用户，当我们以root用户启动supervisord之后。我这里面设置的这个用户，也可以对supervisord进行管理默认情况是不设置。。。非必须设置项
;user=chrism          
;这个参数是supervisord的标识符，主要是给XML_RPC用的。当你有多个supervisor的时候，而且想调用XML_RPC统一管理，就需要为每个supervisor设置不同的标识符了默认是supervisord。。。非必需设置
;identifier=supervisor
;这个参数是当supervisord作为守护进程运行的时候，设置这个参数的话，启动supervisord进程之前，会先切换到这个目录默认不设置。。。非必须设置
;directory=/tmp
; 这个参数当为false的时候，会在supervisord进程启动的时候，把以前子进程产生的日志文件(路径为AUTO的情况下)清除掉。有时候咱们想要看历史日志，当然不想日志被清除了。所以可以设置为true默认是false，有调试需求的同学可以设置为true。。。非必须设置
;nocleanup=true
; 当子进程日志路径为AUTO的时候，子进程日志文件的存放路径。默认路径是这个东西，执行下面的这个命令看看就OK了，处理的东西就默认路径python -c "import tempfile;print tempfile.gettempdir()"非必须设置
;childlogdir=/tmp            
; 这个是用来设置环境变量的，supervisord在linux中启动默认继承了linux的环境变量，在这里可以设置supervisord进程特有的其他环境变量。supervisord启动子进程时，子进程会拷贝父进程的内存空间内容。 所以设置的这些环境变量也会被子进程继承。小例子：environment=name="haha",age="hehe"默认为不设置。。。非必须设置
;environment=KEY="value"     
; 这个选项如果设置为true，会清除子进程日志中的所有ANSI 序列。什么是ANSI序列呢？就是我们的\n,\t这些东西。默认为false。。。非必须设置
;strip_ansi=false            
 
; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections

;这个选项是给XML_RPC用的，当然你如果想使用supervisord或者web server 这个选项必须要开启的
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

;这个主要是针对supervisorctl的一些配置 
[supervisorctl]
; 这个是supervisorctl本地连接supervisord的时候，本地UNIX socket路径，注意这个是和前面的[unix_http_server]对应的默认值就是unix:///tmp/supervisor.sock。。非必须设置serverurl=unix:///tmp/supervisor.sock 
;这个是supervisorctl远程连接supervisord的时候，用到的TCP socket路径 注意这个和前面的[inet_http_server]对应默认就是http://127.0.0.1:9001。。。非必须项
;serverurl=http://127.0.0.1:9001;
; 用户名默认空。。非必须设置    
;username=chris
 ; 密码默认空。。非必须设置
;password=123
;输入用户名密码时候的提示符默认supervisor。。非必须设置
;prompt=mysupervisor         
; 这个参数和shell中的history类似，我们可以用上下键来查找前面执行过的命令 默认是no file的。。所以我们想要有这种功能，必须指定一个文件。。。非必须设置
;history_file=~/.sc_history  

; 下面开始是具体的项目配置
; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;这个就是咱们要管理的子进程了，":"后面的是名字，最好别乱写和实际进程有点关联最好。这样的program我们可以设置一个或多个，一个program就是要被管理的一个进程
;[program:theprogramname]
;这个就是我们的要启动进程的命令路径了，可以带参数例子：/home/test.py -a 'hehe'有一点需要注意的是，我们的command只能是那种在终端运行的进程，不能是
守护进程。这个想想也知道了，比如说command=service httpd start。
httpd这个进程被linux的service管理了，我们的supervisor再去启动这个命令这已经不是严格意义的子进程了。这个是个必须设置的项
;command=/bin/cat
;这个是进程名，如果我们下面的numprocs参数为1的话，就不用管这个参数了，它默认值%(program_name)s也就是上面的那个program冒号后面的名字，但是如果numprocs为多个的话，那就不能这么干了。想想也知道，不可能每个进程都用同一个进程名吧。
;process_name=%(program_name)s ; 
; 启动进程的数目。当不为1时，就是进程池的概念，注意process_name的设置默认为1    。。非必须设置         
;numprocs=1
;进程运行前，会前切换到这个目录默认不设置。。。非必须设置
;directory=/tmp
; 进程掩码，默认none，非必须
;umask=022
; 子进程启动关闭优先级，优先级低的，最先启动，关闭的时候最后关闭,默认值为999 。。非必须设置
;priority=999
; 如果是true的话，子进程将在supervisord启动后被自动启动默认就是true   。。非必须设置
;autostart=true
; 这个是设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的退出码的时候，才会被自动重启。当为true的时候，只要子进程挂掉，将会被无条件的重启
;autorestart=unexpected
; 这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了默认值为1 。。非必须设置
;startsecs=1
; 当进程启动失败后，最大尝试启动的次数。。当超过3次后，supervisor将把此进程的状态置为FAIL默认值为3 。。非必须设置
;startretries=3
; 注意和上面的的autorestart=unexpected对应。。exitcodes里面的定义的退出码是expected的。
;exitcodes=0,2
; 进程停止信号，可以为TERM, HUP, INT, QUIT, KILL, USR1, or USR2等信号默认为TERM 。。当用设定的信号去干掉进程，退出码会被认为是expected 非必须设置
;stopsignal=QUIT
; 这个是当我们向子进程发送stopsignal信号后，到系统返回信息给supervisord，所等待的最大时间。 超过这个时间，supervisord会向该子进程发送一个强制kill的信号。默认为10秒。。非必须设置
;stopwaitsecs=10
;这个东西主要用于，supervisord管理的子进程，这个子进程本身还有子进程。那么我们如果仅仅干掉supervisord的子进程的话，子进程的子进程有可能会变成孤儿进程。所以咱们可以设置可个选项，把整个该子进程的整个进程组都干掉。 设置为true的话，一般killasgroup也会被设置为true。需要注意的是，该选项发送的是stop信号默认为false。。非必须设置。。
;stopasgroup=false
; 这个和上面的stopasgroup类似，不过发送的是kill信号
;killasgroup=false
; 如果supervisord是root启动，我们在这里设置这个非root用户，可以用来管理该program默认不设置。。。非必须设置项
;user=chrism      
; 如果为true，则stderr的日志会被写入stdout日志文件中默认为false，非必须设置
;redirect_stderr=true
; 子进程的stdout的日志路径，可以指定路径，AUTO，none等三个选项。设置为none的话，将没有日志产生。设置为AUTO的话，将随机找一个地方生成日志文件，而且当supervisord重新启动的时候，以前的日志文件会被清空。当 redirect_stderr=true的时候，sterr也会写进这个日志文件
;stdout_logfile=/a/path
; 日志文件最大大小，和[supervisord]中定义的一样。默认为50
;stdout_logfile_maxbytes=1MB   
; 和[supervisord]定义的一样。默认10
;stdout_logfile_backups=10     
; 这个东西是设定capture管道的大小，当值不为0的时候，子进程可以从stdout发送信息，而supervisor可以根据信息，发送相应的event。默认为0，为0的时候表达关闭管道。。。非必须项
;stdout_capture_maxbytes=1MB   
; 当设置为ture的时候，当子进程由stdout向文件描述符中写日志的时候，将触发supervisord发送PROCESS_LOG_STDOUT类型的event默认为false。。。非必须设置
;stdout_events_enabled=false
;这个东西是设置stderr写的日志路径，当redirect_stderr=true。这个就不用设置了，设置了也是白搭。因为它会被写入stdout_logfile的同一个文件中默认为AUTO，也就是随便找个地存，supervisord重启被清空。。非必须设置
;stderr_logfile=/a/path        
;stderr_logfile_maxbytes=1MB   ; 这个出现好几次了，就不重复了
;stderr_logfile_backups=10     ; 这个也是
;stderr_capture_maxbytes=1MB   ; 这个一样，和stdout_capture一样。 默认为0，关闭状态
;stderr_events_enabled=false   ; 这个也是一样，默认为false
;environment=A="1",B="2"       ; 这个是该子进程的环境变量，和别的子进程是不共享的
;serverurl=AUTO                ;


; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername] ;这个东西其实和program的地位是一样的，也是suopervisor启动的子进程，不过它干的活是订阅supervisord发送的event。他的名字就叫listener了。我们可以在listener里面做一系列处理，比如报警等等楼主这两天干的活，就是弄的这玩意
;command=/bin/eventlistener    ; 这个和上面的program一样，表示listener的可执行文件的路径
;process_name=%(program_name)s ; 这个也一样，进程名，当下面的numprocs为多个的时候，才需要。否则默认就OK了
;numprocs=1                    ; 相同的listener启动的个数
;events=EVENT                  ; event事件的类型，也就是说，只有写在这个地方的事件类型。才会被发送 
;buffer_size=10                ; 这个是event队列缓存大小，单位不太清楚，楼主猜测应该是个吧。当buffer超过10的时候，最旧的event将会被清除，并把新的event放进去。默认值为10。。非必须选项

;directory=/tmp                ; 进程执行前，会切换到这个目录下执行默认为不切换。。。非必须
;umask=022                     ; 淹没，默认为none，不说了
;priority=-1                   ; 启动优先级，默认-1，也不扯了
;autostart=true                ; 是否随supervisord启动一起启动，默认true
;autorestart=unexpected        ; 是否自动重启，和program一个样，分true,false,unexpected等，注意unexpected和exitcodes的关系
;startsecs=1                   ; 也是一样，进程启动后跑了几秒钟，才被认定为成功启动，默认1
;startretries=3                ; 失败最大尝试次数，默认3
;exitcodes=0,2                 ; 期望或者说预料中的进程退出码，
;stopsignal=QUIT               ; 干掉进程的信号，默认为TERM，比如设置为QUIT，那么如果QUIT来干这个进程那么会被认为是正常维护，退出码也被认为是expected中的
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ;设置普通用户，可以用来管理该listener进程。
                                默认为空。。非必须设置
;redirect_stderr=true          ; 为true的话，stderr的log会并入stdout的log里面
                                默认为false。。。非必须设置
;stdout_logfile=/a/path        ; 这个不说了，好几遍了
;stdout_logfile_maxbytes=1MB   ; 这个也是
;stdout_logfile_backups=10     ; 这个也是
;stdout_events_enabled=false   ; 这个其实是错的，listener是不能发送event
;stderr_logfile=/a/path        ; 这个也是
;stderr_logfile_maxbytes=1MB   ; 这个也是
;stderr_logfile_backups        ; 这个不说了
;stderr_events_enabled=false   ; 这个也是错的，listener不能发送event
;environment=A="1",B="2"       ; 这个是该子进程的环境变量
                                 默认为空。。。非必须设置
;serverurl=AUTO                ; override serverurl computation (childutils)


; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]  ;这个东西就是给programs分组，划分到组里面的program。我们就不用一个一个去操作了我们可以对组名进行统一的操作。 注意：program被划分到组里面之后，就相当于原来的配置从supervisor的配置文件里消失了。。。supervisor只会对组进行管理，而不再会对组里面的单个program进行管理了
;programs=progname1,progname2  ; 组成员，用逗号分开这个是个必须的设置项
;priority=999                  ; 优先级，相对于组和组之间说的默认999。。非必须选项

 
; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files cannot
; include files themselves.

;[include]                         ;这个东西挺有用的，当我们要管理的进程很多的时候，写在一个文件里面就有点大了。我们可以把配置信息写到多个文件中，然后include过来
;files = relative/directory/*.ini

```

配置文件都有说明，且很简单，就不做多的描述了，在上面有一些建议修改的目录，若做了修改，则应先创建这些文件，需要注意权限问题，很多错误都是没有权限造成的。

P.S. 需要重点关注的是以部分

[program:x]中配置要监控的进程

### 启动服务端

现在，让我们来启动supervisor服务。

```
[root@iZwz99i40ignvwjuinf024Z ~]# supervisord -c /etc/supervisord.conf
```

如果不指定 `-c` 选项，那么会使用以下顺序路径进行检索 `supervisord.conf` 文件

```bash
$CWD/supervisord.conf
$CWD/etc/supervisord.conf
/etc/supervisord.conf
/etc/supervisor/supervisord.conf (since Supervisor 3.3.0)
../etc/supervisord.conf (Relative to the executable)
../supervisord.conf (Relative to the executable)
```

查看supervisord 是否运行：

```bash
[root@iZwz99i40ignvwjuinf024Z ~]# ps aux|grep superviosrd
output:xxxx   82039      1  0 11:22 ?        00:00:00 /usr/local/bin/python /usr/local/bin/supervisord -c /etc/supervisord.conf
```

查看端口是否被监听 或者使用netstat,扩展阅读 à  [linux 命令 -- netstat 和 lsof](https://www.cnblogs.com/GODYCA/archive/2013/02/27/2935606.html)

```bash
[root@iZwz99i40ignvwjuinf024Z www.zsxiaogan.cn]# lsof -i:10001
COMMAND     PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
superviso 25438 root    4u  IPv4 1487990      0t0  TCP *:scp-config (LISTEN)
```

### 项目配置及运行

上面我们已经把 supervisrod 运行起来了，现在可以添加我们要管理的进程的配置文件。可以把所有配置项都写到 supervisord.conf 文件里，但并不推荐这样做，而是通过 include 的方式把不同的程序（组）写到不同的配置文件里，对，就是默认配置中的最后的那个include。下面来对项目进行简单的配置。

假设我们把项目配置文件放在这个目录中:/etc/supervisor/

则我们需要修改/etc/supervisord.conf 中的include为：

```ini
[include] 
files = /etc/supervisor/*.conf
```

以下为一个博文中的配置文件目录：

/etc/supervisor/update_ip.conf

具体配置为

```ini
[program:update_ip] ;项目名称
directory = /home/xxxx/works/ip_update/ip_update_on_server_no_1/ ; 程序的启动目录
command = python /home/xxxx/works/ip_update/ip_update_on_server_no_1/update_ip_internal.py  ; 启动命令，可以看出与手动在命令行启动的命令是一样
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = shimeng          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 50MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /home/xxxx/works/ip_update/ip_update_on_server_no_1/supervisor.log
loglevel=info

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket

[unix_http_server]
file=/tmp/supervisor.sock   ; (the path to the socket file)
chmod=0777                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=shimeng              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[inet_http_server]         ; inet (TCP) server disabled by default
port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
username=shimeng              ; (default is no username (open server))
password=123
```

**配置详解：**

a)  在supervisord.conf文件中，分号“；”后面的内容表示注释

b)  [group:组名]，设置一个服务分组，programs后面跟组内所有服务的名字，以分号分格。

c)  [program：服务名]，下面是这个服务的具体设置：

>Command:启用Tornado服务文件的命令，也就是我们手动启动的命令。
>Directory:服务文件所在的目录
>User:启用服务的用户
>Autorestart:是否自动重启服务
>stdout_logfile：服务的产生的日起文件
>loglevel:日志级别

配置完成以后，即可运行：

supervisord -c /etc/supervisord.conf

查看运行状态

```bash
$ supervisorctl status
out:
update_ip         RUNNING   pid 62040, uptime 0:10:09
```

打开浏览器，输入127.0.0.9001,输入用户名与密码（如果配置文件中inet_http_server中作了设置），可以看到下面这个界面：

![image](file:///C:/Users/sheny/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

### 使用supervisorctl

在启动服务之后，运行：

```bash
supervisorctl -c /etc/supervisord.conf
out:
update_ip         RUNNING   pid 62040, uptime 0:10:09
```

若成功，则会进入supervisorctl的shell界面，有以下方法：

```
status    # 查看程序状态
stop update_ip   # 关闭 update_ip 程序
start update_ip  # 启动 update_ip 程序
restart update_ip    # 重启 update_ip 程序
reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
update    ＃ 重启配置文件修改过的程序
```

执行相关操作后，可以在web端看到具体的变化情况，如stop 程序

stop update_ip

![image](file:///C:/Users/sheny/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

其实，也可以不使用supervisorctl shell界面，而在bash终端运行：

```
supervisorctl status
supervisorctl stop usercenter
supervisorctl start usercenter
supervisorctl restart usercenter
supervisorctl reread
supervisorctl update 
```

### 多个进程管理

按照官方文档的定义，一个 [program:x] 实际上是表示一组相同特征或同类的进程组，也就是说一个 [program:x] 可以启动多个进程。这组进程的成员是通过 numprocs 和 process_name 这两个参数来确定的，这句话什么意思呢，我们来看这个例子。

```ini
; 设置进程的名称，使用 supervisorctl 来管理进程时需要使用该进程名
[program:foo] 
; 可以在 command 这里用 python 表达式传递不同的参数给每个进程
command=python server.py --port=90%(process_num)02d
directory=/home/python/tornado_server ; 执行 command 之前，先切换到工作目录
; 若 numprocs 不为1，process_name 的表达式中一定要包含 process_num 来区分不同的进程
numprocs=2                   
process_name=%(program_name)s_%(process_num)02d; 
user=oxygen                 ; 使用 oxygen 用户来启动该进程
autorestart=true  ; 程序崩溃时自动重启
redirect_stderr=true      ; 重定向输出的日志
stdout_logfile = /var/log/supervisord/
tornado_server.log
loglevel=info
```

上面这个例子会启动两个进程，process_name 分别为 foo:foo_01 和 foo:foo_02。通过这样一种方式，就可以用一个 [program:x] 配置项，来启动一组非常类似的进程。

更详细配置，点击[这里](http://supervisord.org/configuration.html#program-x-section-settings)

Supervisor 同时还提供了另外一种进程组的管理方式，通过这种方式，可以使用 supervisorctl 命令来管理一组进程。跟 [program:x] 的进程组不同的是，这里的进程是一个个的 [program:x] 。

```ini
[group:thegroupname]
programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
priority=999                  ; the relative start priority (default 999)
```

当添加了上述配置后，progname1 和 progname2 的进程名就会变成 thegroupname:progname1 和 thegroupname:progname2 以后就要用这个名字来管理进程了，而不是之前的 progname1。
以后执行 supervisorctl stop thegroupname: 就能同时结束 progname1 和 progname2，执行 supervisorctl stop thegroupname:progname1 就能结束 progname1。

## 结尾

实际上，默认情况下，supervisored 也是一个进程，最理想的的情况应该是将其安装为系统服务，安装方法可以参考[这里](http://serverfault.com/questions/96499/how-to-automatically-start-supervisord-on-linux-ubuntu),安装脚本参考[这里](https://github.com/Supervisor/initscripts)

### 设置开机启动supervisor

1. 新建服务的方式

新建开机启动服务

```bash
vim  /lib/systemd/system/supervisord.service
```

在supervisord.service中添加以下内容：

```ini
# supervisord service for systemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)

[Unit]
Description=Supervisor daemon[Service]
Type=forking
ExecStart=/usr/bin/supervisord
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

**注意：**如果的supervisord的安装目录是其他目录,比如/usr/local/bin/supervisord,就需要把上边目录修改为安装目录

修改/etc/supervisord.conf配置文件：

```bash
$ vim /etc/supervisrod.conf
# 找到nodaemon=false 改成true
```

将服务脚本添加到systemctl自启动服务：

```bash
systemctl enable supervisord.service
```

重启系统测试开机启动。

或者

不重启测试
 systemctl enable supervisord
 验证一下是否为开机启动：systemctl is-enabled supervisord

2. 配置 /etc/rc.local

centos7以上不太建议,详见补充说明.

Linux 在启动的时候会执行 /etc/rc.local 里面的脚本，可以在这里添加执行命令

```
# 如果是 Ubuntu 添加以下内容
/usr/local/bin/supervisord -c /etc/supervisord.conf
# 如果是 Centos 添加以下内容
/usr/bin/supervisord -c /etc/supervisord.conf
```

以上内容需要添加在 exit 命令前，而且由于在执行 rc.local 脚本时，PATH 环境变量未全部初始化，因此命令需要使用绝对路径。

在添加前，先在终端测试一下命令是否能正常执行，如果找不到 supervisord，可以用如下命令找到

```shell
sudo find / -name supervisord
output:
/usr/local/bin/supervisord
```

## 补充

1. Supervisor还有事件响应，写XML-RPC扩展等高级功能，这里就不细述了，大家可以去学习下。

2. 在查资料的时候,发现centos下/etc/rc.local,文件是干什么的.在centos7之前应该是可以直接在这个文件里面配置开机启动项的,centos7以后就开始废弃这个东西,但是现在还是可以使用的

   最近发现**centos7** 的**/etc/rc.local**不会开机执行，于是认真看了下**/etc/rc.local**文件内容的就发现了问题的原因了

   ```
   #!/bin/bash
   # THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
   #
   # It is highly advisable to create own systemd services or udev rules
   # to run scripts during boot instead of using this file.
   #
   # In constrast to previous versions due to parallel execution during boot
   # this script will NOT be run after all other services.
   #
   # Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
   # that this script will be executed during boot.
   ```

    **翻译：**

   ```
   #这个文件是为了兼容性的问题而添加的。
   #
   #强烈建议创建自己的systemd服务或udev规则来在开机时运行脚本而不是使用这个文件。
   #
   #与以前的版本引导时的并行执行相比较，这个脚本将不会在其他所有的服务后执行。
   #
   #请记住，你必须执行“chmod +x /etc/rc.d/rc.local”来确保确保这个脚本在引导时执行。
   ```

   于是我有确认了下**/etc/rc.local**的权限

   ```
   [root@localhost ~]# ll /etc/rc.local
   lrwxrwxrwx. 1 root root 13 8月 12 06:09 /etc/rc.local -> rc.d/rc.local
   [root@localhost ~]# ll /etc/rc.d/rc.local
   -rw-r--r--. 1 root root 477 6月 10 13:35 /etc/rc.d/rc.local
   ```

   /etc/rc.d/rc.local没有执行权限，于是按说明的内容执行

   **chmod +x /etc/rc.d/rc.local**

   重启后发现/etc/rc.local能够执行了。

   看样子是版本的变迁，/etc/rc.local /etc/rc.d/rc.local正在弃用的路上。

3. 报错 unix:///var/run/supervisor.sock no such file

   如果没有修改supervisord.pid 以及 supervisor.sock的默认配置,当linux清理了tmp文件之后就会报这个错误,有两种方式解决,

   一 修改配置直接重启

   二 手动建立一个sock文件

   ```bash
   sudo touch /var/tmp/supervisor.sock
   sudo chmod 777 /var/tmp/supervisor.sock
   sudo service supervisor restart
   ```

## 参考

>[Supervisor的官方文档](http://www.supervisord.org/index.html)
>
>[Linux进程管理工具supervisor安装及使用](https://blog.csdn.net/rapier512/article/details/80541579)
>
>[进程管理利器-supervisor部署记录](https://www.cnblogs.com/kevingrace/p/7525200.html)
>
>[Supervisor安装与配置（Linux/Unix进程管理工具）](https://blog.csdn.net/xyang81/article/details/51555473)
