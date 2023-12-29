# 安装和配置nginx（非docker）

> Nginx 是开源、高性能、高可靠的 Web 和反向代理服务器，而且支持热部署，几乎可以做到 7 * 24 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。性能是 Nginx 最重要的考量，其占用内存少、并发能力强、能支持高达 5w 个并发连接数，最重要的是，配置使用也比较简单。

因为我们用nginx作Web服务器，所以我们需要先安装nginx服务。

#### 卸载nginx

参考：https://www.python100.com/html/0S5UG8L956FT.html

```
查看版本
nginx -v
```

#### 安装(CentOS)

具体步骤如下：

使用root用户远程登录阿里云服务器，使用yum命令进行安装。

```
# 安装nginx的依赖环境
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

# 下载nginx安装包。
wget -c https://nginx.org/download/nginx-1.10.1.tar.gz

# 将安装包解压到/usr/local目录下
tar -xvf nginx-1.10.1.tar.gz -C /usr/local

# 进入/usr/local目录，确认nginx解压到该目录下
cd /usr/local
ls

# 进入nginx-1.10.1目录，会发现该目录下有一个configure文件，执行该配置文件
cd nginx-1.10.1/
ls
./configure

# 编译并安装nginx。
make
make install

# 查找nginx安装目录
whereis nginx
# 或者
find / -name nginx.conf

# 进入nginx的【安装目录】
cd /usr/local/nginx
ls

# 由于nginx默认通过80端口访问，而Linux默认情况下不会开发该端口号，因此需要开放linux的80端口供外部访问
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT

# 进入安装目录（usr/local/nginx）的sbin 目录下，启动nginx。
cd sbin
./nginx

# 没有任何消息，代表启动成功。此时，便可以通过“公网IP+端口”的方式访问 http://xx.xx.xxx.xxx:80/ 进入nginx欢迎页面了
```

#### 配置nginx转发

如果想对这个项目用特定的子域名进行访问，就要先进行 域名解析，比如我这里是用阿里的

![image-20231221150721215](http://biji.51automate.cn/blogs/img/202312211507579.png)

![image-20231221150904872](http://biji.51automate.cn/blogs/img/202312211509862.png)

```
# 专门为项目创建一个【前端部署目录】
mkdir -p /home/www/fastapivue2

在nginx配置文件中，配置server服务（监听端口和项目部署的目录）
配置文件路径如下：/usr/local/nginx/conf/nginx.conf

修改完之后，重新加载配置文件，才生效
/usr/local/nginx/sbin/nginx -s reload
```

配置文件 `nginx.conf`, 配置文件内容如下：直接复制即可

```
#user  nobody;
worker_processes  auto;  # Nginx 进程数，一般设置为和 CPU 核数一样

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

# 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
http {
    
    
    sendfile        on;     # 开启高效传输模式
    keepalive_timeout  65;  # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;
    
    include       mime.types;  # 文件扩展名与类型映射表
    default_type  application/octet-stream;   # 默认文件类型

    server {
        listen       80;
        # 如果没有备案域名，可以填写服务器公网ip，以后有了再改回来
        server_name  51automate.cn;

        location / {
            root    /home/www/hexo;  # 网站根目录
            index  index.html index.htm;   # 默认首页文件
            # deny 172.168.22.11; # 禁止访问的ip地址，可以为all
            # allow 172.168.33.44；# 允许访问的ip地址，可以为all
        }
    }

    server {
        listen       80;
        server_name  myfav.51automate.cn;

        location / {
            root    /home/www/myFavorite;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
        
        # 访问链接中出现 /api/ 就转发到本地的9090端口（该端口是 后端服务端口）
        location /api/ {
            proxy_pass   http://127.0.0.1:9090/;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       80;
        server_name  fv2.51automate.cn;

        location / {
            root    /home/www/fastapivue2;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
        
        # 访问链接中出现 /api/ 就转发到本地的8888端口（该端口是 后端服务端口）
        location /api/ {
            proxy_pass   http://127.0.0.1:8888/api/;
        }
    }
    
}

```



重点注意端口转发内容

```
location /api/ {
            proxy_pass   http://127.0.0.1:8888/;
        }
```

另外配置nginx之后要**重新加载**才生效。



<img src="http://biji.51automate.cn/blogs/img/202312291356737.png"/>

#### 拓展：nginx的其它常用命令

```
重启nginx
/usr/local/nginx/sbin/nginx -s reopen
停止服务
/usr/local/nginx/sbin/nginx -s stop

手动启动nginx服务
cd /usr/local/nginx       进入nginx安装目录
sudo /usr/local/nginx/sbin/nginx  管理员权限【启动nginx】

---------------------------------------------------------------

# 开机配置
# 开机自动启动
systemctl enable nginx
# 关闭开机自动启动
systemctl disable nginx


# 启动Nginx
# 启动Nginx成功后，可以直接访问主机IP，此时会展示Nginx默认页面
systemctl start nginx

# 停止Nginx
systemctl stop nginx

# 重启Nginx
systemctl restart nginx

# 重新加载Nginx
systemctl reload nginx

# 查看 Nginx 运行状态
systemctl status nginx

# 查看Nginx进程
ps -ef | grep nginx

# 杀死Nginx进程
# 根据上面查看到的Nginx进程号，杀死Nginx进程，-9 表示强制结束进程
kill -9 pid

# Nginx 应用程序命令：
# 向主进程发送信号，重新加载配置文件，热重启
nginx -s reload
# 重启 Nginx
nginx -s reopen
# 快速关闭
nginx -s stop
# 等待工作进程处理完成后关闭
nginx -s quit
# 查看当前 Nginx 最终的配置
nginx -T
# 检查配置是否有问题
nginx -t
```

##### 查看端口下的服务

```
# 查看占用端口号的服务
netstat -nlp

# 查看某个服务的进程信息
ps -ef|grep 服务名称

# 查看当前进程的端口号
netstat -anp|grep 进程号

# 查看某个端口下的服务
lsof -i:端口号
lsof -i:端口号|grep 服务名
```



![image-20231229135648546](http://biji.51automate.cn/blogs/img/202312291356737.png)