# Docker安装Nginx，配置宿主机和容器的工作目录挂载和多个端口监听等



Docker安装Nginx，配置宿主机和容器的工作目录挂载和多个端口监听等

https://www.cnblogs.com/haoxianrui/p/13591429.html

### Docker安装nginx

##### 下载nginx镜像

```
docker pull nginx
```

##### 查看下载的nginx镜像

```
docker images
```

##### 根据镜像 创建 一个容器

```
docker run -p 9091:80 --name nginx -d nginx
--name  #给你启动的容器起个名字，以后可以使用这个名字启动或者停止容器
-p #映射端口，将docker宿主机的9091端口和容器的80端口进行绑定
-v #挂载文件用的
-d #表示启动的是哪个镜像。

```

![image-20231226153112323](http://biji.51automate.cn/blogs/img/202312261531404.png)

### Nginx服务的配置和部署

为什么需要做文件的映射？

> 我们在使用容器的过程中需，有时候需要对容器中的文件进行修改管理，如果不做文件映射的化，我们使用docker exec -it 容器ID/容器名 /bin/bash 才能进入nginx中的文件里面如图
>
> ![image-20231226135448771](http://biji.51automate.cn/blogs/img/202312261354397.png)
>
> 如果把关键文件映射到主机上，那么就可以在主机中进行修改而不必进入文件当中才进行修改了。

##### 将nginx的配置文件、日志目录映射到宿主机

先在宿主机上 新建目录，用于存储映射文件。

```
mkdir -p /home/docker/nginx/{conf,conf/conf.d,log,www}
```

> ```
> 宿主机                                   docker
> /home/docker/nginx/www                /usr/share/nginx/html #网页文件
> /home/docker/nginx/conf/nginx.conf    /etc/nginx/nginx.conf#配置文件
> /home/docker/nginx/logs               /var/log/nginx#日志文件
> ```



```
# 复制容器内文件到宿主机指定目录
docker cp nginx:/etc/nginx/nginx.conf /home/docker/nginx/conf/nginx.conf
docker cp nginx:/etc/nginx/conf.d/default.conf /home/docker/nginx/conf/conf.d/
docker cp nginx:/usr/share/nginx/html/index.html /home/docker/nginx/www/

    
# 把先前的nginx容器 停止并删除
docker stop nginx
docker rm nginx

# 运行nginx，同时进行文件挂载
docker run -p 9091:80 --name nginx \
    -v /home/docker/nginx/www:/usr/share/nginx/html \
    -v /home/docker/nginx/log:/var/log/nginx \
    -v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /home/docker/nginx/conf/conf.d/:/etc/nginx/conf.d/ \
    -d nginx
        
# 修改配置文件内容并重新加载 Nginx 的配置文件（以下命令二选一即可）
docker exec -it nginx nginx -s reload
docker kill -s HUP nginx


```

### Nginx配置

通过nginx为**多个域名**配置服务，使得通过浏览器可以访问到相应的域名。

##### 创建 不同的 域名文件夹

- 在映射到主机下的`/home/docker/nginx/www`下【**域名的文件夹**】

  ```
  mkdir -p /home/docker/nginx/www/{www.51automate.cn,fv2.51automate.cn,myfav.51automate.cn,blog.51automate.cn}
  ```

  

- 新建我的配置文件 `my.conf`

  ```
  touch  /home/docker/nginx/conf/conf.d/my.conf
  ```

  配置内容如下：

  ```
  
  server {
      listen       80;
      server_name  myfav.51automate.cn;
      location / {
          root   /usr/share/nginx/html/myfav.51automate.cn;
          index  index.html index.htm;
          try_files $uri $uri/ /index.html;
      }
      
      location /api/ {
          proxy_pass   http://www.51automate.cn:9090/;
          # proxy_pass   http://127.0.0.1:9090/;
      }
  
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
  }
  
  server {
      listen       80;                    # 注意这里nginx 监听的是容器内的端口（我们启动的容器就将容器内的80端口 映射到 宿主机的其它端口）
      server_name  fv2.51automate.cn;
  
      #access_log  /var/log/nginx/host.access.log  main;
      
      location / {
          root   /usr/share/nginx/html/fv2.51automate.cn; # 注意 nginx的工作目录 是容器内 前端文件的根目录，而非宿主机目录
          index  index.html index.htm;  # 前端主页
          try_files $uri $uri/ /index.html;
      }
      
      # 访问链接中出现 /api/ 就转发到本地的8888端口（该端口是 后端服务端口）
      location /api/ {
          proxy_pass   http://www.51automate.cn:8888/api/;  # ip+端口 或者 域名+端口，不可使用 localhost 或者 127.0.0.1。原因：容器是一个独立的运行环境，容器内部的 localhost 地址指向的是容器本身，而不是宿主机。而我们的后端代码是运行在宿主机上。
          # proxy_pass   http://127.0.0.1:8888/;  # 不可使用
      }
  
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
  
  }
  
  
  ```

  配置内容这里重点强调2点 

  1.  root的路径    ：  nginx的工作目录 是容器内 前端文件的根目录，而**非宿主机目录**

     ```
     location / {
             root   /usr/share/nginx/html/fv2.51automate.cn; # 注意 nginx的工作目录 是容器内 前端文件的根目录，而非宿主机目录
             index  index.html index.htm;  # 前端主页
             try_files $uri $uri/ /index.html;
         }
     ```

  2. proxy_pass  配置 。正确的写法是： ip+端口 或者 域名+端口，不可使用 localhost 或者 127.0.0.1。

     > 原因：容器是一个独立的运行环境，容器内部的 localhost 地址指向的是容器本身，而不是宿主机, 而我们的后端代码是运行在宿主机上。

- 试试效果

  ![image-20231226153655146](http://biji.51automate.cn/blogs/img/202312261536494.png)

  ![image-20231226153747980](http://biji.51automate.cn/blogs/img/202312261537189.png)

- vue前端项目打包文件 放在 对应的 域名文件下，即可完成部署。

- 默认情况下 我们需要宿主机的`80端口`，免得用户每次都要输入一个端口，很不友好。

  ```
  # 把先前的nginx容器 停止并删除
  docker stop nginx
  docker rm nginx
  
  # 运行nginx，同时进行文件挂载
  docker run -p 80:80 --name nginx \
      -v /home/docker/nginx/www:/usr/share/nginx/html \
      -v /home/docker/nginx/log:/var/log/nginx \
      -v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
      -v /home/docker/nginx/conf/conf.d/:/etc/nginx/conf.d/ \
      -d nginx
          
  # 修改配置文件内容并重新加载 Nginx 的配置文件（以下命令二选一即可）
  docker exec -it nginx nginx -s reload
  docker kill -s HUP nginx
  ```

  重新访问 `fv2.51automate.cn` 搞定。

  

上面配置文件的内容 有这样一段 

```
# 访问链接中出现 /api/ 就转发到本地的8888端口（该端口是 后端服务端口）
    location /api/ {
        proxy_pass   http://127.0.0.1:8888/api/;
```

可以用来解决前端跨域问题；也可进行反向代理。

到此普通用户已经够用了，更多信息可参考：https://blog.51cto.com/u_14020077/5836696