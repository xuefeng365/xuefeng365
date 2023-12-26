# 进程管理工具supervisor守护进程

supervisor是一个进程管理工具，监控进程状态，异常退出时能自动重启。同时有web页面交互比较好

> Windows系统无法使用

```
# 安装
yum install -y supervisor
```

使用`yum`会生成默认配置`/etc/supervisord.conf`和目录`/etc/supervisord.d`

在`/etc/supervisord.d`的目录下创建`conf`和`log`两个目录，`conf`用于存放管理进程的配置，`log`用于存放管理进程的日志。

```
cd /etc/supervisord.d
mkdir conf log
```

### 修改配置文件内容

#### supervisord.conf的配置

![image-20231226222557761](http://biji.51automate.cn/blogs/img/202312262226005.png)

![image-20231226222717938](http://biji.51automate.cn/blogs/img/202312262227201.png)

![image-20231226224222335](http://biji.51automate.cn/blogs/img/202312262242433.png)

1. 修改`/etc/supervisord.conf`的`[include]`部分，即载入`/etc/supervisord.d/conf`目录下的所有配置。

```
files = supervisord.d/conf/*.conf
```

2. 修改`inet_http_server`

   ```
   
   [inet_http_server]         ; inet (TCP) server disabled by default
   port=*:9001        ; (ip_address:port specifier, *:port for all iface)
   username=root            ; (default is no username (open server))
   password=0             ; (default is no password (open server))
   ```

   修改后重新加载配置文件

   ```
   supervisorctl reload
   ```

   就可以通过 web页面访问 `http://www.51automate.cn:9001`

   ![image-20231226224105027](http://biji.51automate.cn/blogs/img/202312262241262.png)

#### 管理应用的配置

进入到`/etc/supervisord.d/conf`目录，创建管理应用的配置，可以创建多个应用配置。

例如，创建`nginx.conf`配置。

> 守护进程： 在我们生产环境的时候，有些任务是不能停止的，否则业务就会受到影响，那么如何保证这些任务的高可用呢？那就需要用到我们的守护进程了，比方说我们的进程运行挂掉之后自动恢复等等

```
[program:nginx] #  设置进程的名称，使用 supervisorctl 来管理进程时需要使用该进程名 我这里就叫做nginx了!
command=/usr/sbin/nginx -g 'daemon off;' # 需要执行的命令
directory=/etc/nginx # 命令执行的目录或者说执行 command 之前，先切换到工作目录 可以理解为在执行命令前会切换到这个目录 在我这基本没啥用
autostart=true #是否自动启动
autorestart=true #程序意外退出是否自动重启

redirect_stderr=true # 如果为true，则stderr的日志会被写入stdout日志文件中  理解为重定向输出的日志

priority=10 # 启动优先级
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/data/logs/supervisord/nginx.log # 子进程的stdout的日志路径 输出日志文件

stderr_logfile=/data/logs/supervisord/nginx.err.log # 错误日志文件 当redirect_stderr=true。这个就不用

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

#### 修改后更新 `Supervisor`

```
supervisorctl reread # 重新读取配置
supervisorctl update # 更新配置
supervisorctl restart nginx  # 重启 nginx
killall nginx  # 杀掉所有的 nginx 进程. 已经杀不死了 说明守护进程配置成功

supervisorctl status         # 查看一下任务 ok
```

### 常用命令

```
systemctl start supervisord.service     //启动supervisor并加载默认配置文件
systemctl enable supervisord.service    //将supervisor加入开机启动项
systemctl enable supervisord            //将supervisor加入开机启动项
```

### 配置文件详细说明

```
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

;[inet_http_server]         ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user              ;登录管理后台的用户名
;password=123               ;登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord

; [program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程

;包含其它配置文件
[include]
files = relative/directory/*.ini    ;可以指定一个或多个以.ini结束的配置文件
```



---

