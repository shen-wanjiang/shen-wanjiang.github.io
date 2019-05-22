---
layout:     post
title:      进程管理之supervisor的配置和使用
subtitle:   supervisor配置及使用
date:       2019-05-21
author:     WJ
header-img: img/post-bg-python-unicode.jpg
catalog: true
tags:
    - other
---

>[Supervisor](http://supervisord.org)是用Python开发的一个client/server服务，是Linux/Unix系统下的一个进程管理工具，不支持Windows系统。它可以很方便的监听、启动、停止、重启一个或多个进程。用Supervisor管理的进程，当一个进程意外被杀死，supervisort监听到进程死后，会自动将它重新拉起，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。

[GitHub项目](https://github.com/Supervisor/supervisor)

Supervisor是python2写就的一款强大的运维工具,当然,现在已经完全支持python3了,我们以下的操作都是处于python3环境
### 1. 安装
我当前使用的环境是Ubuntu, 推荐使用`apt-get`来安装.
```shell
sudo apt-get install supervisor
```
1. 使用这种操作不需要像原来一样自己导出来配置文件,会自己生成配置文件. 配置文件位置: `/etc/supervisor/supervisor.conf`
2. 如果没有自己生成,就手动生成supervisor的配置文件
运行`sudo echo_supervisord_conf > /etc/supervisor/supervisord.conf`来产生设置

同时也会在`/etc/supervisor`下创建一个`conf.d`的文件夹,来放置后面你要管理的进程配置项, 这个在下面会说道. 如果没有个文件夹,就手动创建一个同名的文件夹

**注意点**
这是用sudo安装的,每次里面的操作需要root权限. 
解决方法: 你可以修改配置文件,改变其权限,这样调用的时候就不用root权限了
在配置文件(`/etc/supervisor/supervisord.conf`)内的 `[unix_http_server]`段落中将chmod修改为0766.这样就不需要root了

当然,你也可以用python的包管理工具来安装(`pip install supervisor`), 这样不用root. 但是pip的安装只能基于python2, 所以还是用`apt-get`来安装

### 2. 卸载
如果想要卸载,使用命令`sudo apt-get remove supervisor`
它会卸载`/usr/bin/`下安装的程序,但是不会删除配置和启动文件,即`/etc/supervisor/`下文件
如果想要查看,文件的位置,可以使用 `whereis supervisor`来查看supervisor安装到了哪些位置

### 3. supervisor配置文件参数说明
supervisor的配置参数较多，下面介绍一下常用的参数配置，详细的配置及说明，请参考[官方文档](http://supervisord.org/)介绍。
注：分号（;）开头的配置表示注释
```shell
[unix_http_server]
file=/tmp/supervisor.sock  ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700         ;socket文件的mode，默认是0700
;chown=nobody:nogroup    ;socket文件的owner，格式：uid:gid
 
;[inet_http_server]     ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001    ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user       ;登录管理后台的用户名
;password=123        ;登录管理后台的密码
 
[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB    ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10      ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info        ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false        ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024         ;可以打开的文件描述符的最小值，默认 1024
minprocs=200         ;可以打开的进程数的最小值，默认 200
 
[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord
 
; [program:tomcat]是被管理的进程配置参数，tomcat是进程的名称
[program:tomcat]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run ; 程序启动命令
autostart=true    ; 在supervisord启动的时候也自动启动
startsecs=10     ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true   ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3    ; 启动失败自动重试次数，默认是3
user=tomcat     ; 用哪个用户启动进程，默认是root
priority=999     ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20  ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false   ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false   ;默认为false，向进程组发送kill信号，包括子进程
 
;包含其它配置文件,这里就是我们所需要配置的文件
[include]
files = /etc/supervisor/conf.d/*.conf    ;可以指定一个或多个以.conf结束的配置文件,有的是以*.ini 结尾的，这个是自定义的 不影响；
```

**为什么要有[include ]呢?**
当我们要监控的程序进程比较多的时候，如果都像tomcat一样都写在此配置文件里的话，那会管理起来很不方便，也很凌乱！所以此时用[include]来管理其他程序进程的配置的文件就非常有必要了
进入`/etc/supervisor/`下的conf.d文件夹. 然后创建以conf结尾的文件作为配置文件: `touch tomcat.conf` 
编辑tomcat.conf:
```shell
[program:tomcat]
command=/usr/local/tomcat/bin/catalina.sh run
user=root
autorstart=true
autorestart=true
startsecs=10
startretries=3
redirect_stderr=true
stdout_logfile=/usr/local/tomcat/logs/log.txt
stderr_logfile=/usr/local/tomcat/logs/err.txt
environment=LC_ALL="C.UTF-8",LANG="C.UTF-8"
```

### 4. 配置启动进程文件参数说明
```shell
#项目名
[program:blog]
#脚本目录
directory=/opt/bin
#脚本执行命令, 可以带参数
command=/usr/bin/python /opt/bin/test.py
#supervisor启动的时候是否随着同时启动，默认True
autostart=true
#当程序exit的时候，这个program不会自动重启,默认unexpected
#设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的
autorestart=false
#这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了。默认值为1
startsecs=1
#日志输出 
stderr_logfile=/tmp/blog_stderr.log 
stdout_logfile=/tmp/blog_stdout.log 
#脚本运行的用户身份 
user = zhoujy 
#把 stderr 重定向到 stdout，默认 false
redirect_stderr = true
#stdout 日志文件大小，默认 50MB
stdout_logfile_maxbytes = 20M
#stdout 日志文件备份数
stdout_logfile_backups = 20


[program:zhoujy] #说明同上
directory=/opt/bin 
command=/usr/bin/python /opt/bin/zhoujy.py 
autostart=true 
autorestart=false 
stderr_logfile=/tmp/zhoujy_stderr.log 
stdout_logfile=/tmp/zhoujy_stdout.log 
#user = zhoujy

```
**注意**
如果你使用了虚拟环境,一定要找对该虚拟环境中python解释器的位置,然后执行.
在上文的例子中,command后面的解释器要找对.


### 5. 启动supervisor服务
`supervisord -c /etc/supervisor/supervisord.conf`

详细解释:
`cd /usr/local/supervisor-3.3.4/supervisor` 可以发现 supervisord.py,这是supervisord的启动程序.
bash环境下可以直接启动,执行命令: `./supervisord.py` 也可以`python supervisord.py`这样启动，
这两种方式不管哪种在不指定配置文件（supervisord.conf)路径的情况下都会有警告的信息.
用户警告：监督管理是作为根运行的，它在默认位置（包括它当前的工作目录）中搜索它的配置文件;您可能想要指定一个"-c"参数，指定配置文件的绝对路径以提高安全性。
所以执行supervisord.py的时候最好加上配置文件路径: `python supervisord.py -c /etc/supervisor/supervisord.conf` 或者直接执行：`supervisord -c /etc/supervisor/supervisord.conf`都可以

### 6. 控制进程
1. 交互终端
    supervisord启动成功后，可以通过supervisorctl客户端控制进程，启动、停止、重启。运行supervisorctl命令，不加参数，会进入supervisor客户端的交互终端，并会列出当前所管理的所有进程。 
    ![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/supervisorctl.png)

    输入help可以查看可以执行的命令列表，如果想看某个命令的作用，运行help 命令名称，如：help stop
    ```shell
    stop vtime_admin // 表示停止vtime_admin进程
    stop all   // 表示停止所有进程
    // ...
    ```

2.  bash终端
```shell
supervisorctl status  # 开始所有进程
supervisorctl stop vtime_admin  # 停止vtime_admin进程
supervisorctl start vtime_admin  # 开始vtime_admin进程
supervisorctl restart vtime_admin  # 重启vtime_admin进程
supervisorctl restart all  # 重启所有的进程
supervisorctl reread  # 读取有更新（增加）的配置文件，不会启动新添加的程序
supervisorctl update  # 重启配置文件修改过的程序
```

3. Web管理界面
![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/supervisor_control.png)

出于安全考虑，默认配置是没有开启web管理界面，需要修改supervisord.conf配置文件打开http访权限，将下面的配置：
```shell
;[inet_http_server]     ; inet (TCP) server disabled by default
;port=127.0.0.1:9001    ; (ip_address:port specifier, *:port for all iface)
;username=user       ; (default is no username (open server))
;password=123        ; (default is no password (open server))
```
修改为:
```
[inet_http_server]     ; inet (TCP) server disabled by default
port=0.0.0.0:9001     ; (ip_address:port specifier, *:port for all iface)
username=user       ; (default is no username (open server))
password=123        ; (default is no password (open server))
```
- port：绑定访问IP和端口，这里是绑定的是本地IP和9001端口 
- username：登录管理后台的用户名 
- password：登录管理后台的密码


就是一个简单的进程管理软件. 启动的命令一定到找对,就可以了(虚拟环境中的变量),而不用其它设置其它的变量. 一定要注意配置文件的方法, 切记,不然根本启动不了
现在新的安装包,不需要像原来一样自己导出配置文件,就自己生成了配置文件(位置在/etc/supervisor/supervisor.conf),所以,直接在那个里面编辑就好了. 这是用sudo apt-get install supervisor 安装的,里面的进行操作的动画需要sudo权限. 如果是卸载 ,sudo apt-get remove supervisor 会卸载/usr/bin/ 下的文件,但是不会删除 /etc/supervisor/下的启动文件

supervisor启动后的进程应该会自己更新,你如果git pull 拉去新代码,针对supervisorctl 中不用restart all 就可以将新拉去的代码更新(fabric)