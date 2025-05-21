# 爬虫系列：Python代理IP池项目构建成镜像文件并上传到Docker hub

为什么要上传到Docker hub

> 随时随地 只需要 一条命令 `docker pull spider_proxy_pool:v1`  即可下载镜像

为什么要镜像？

> 只需确定好映射端口 一条命令即可快速启动容器，ip 随时取用 

```
docker run --env DB_CONN=redis://:@ip:6379/0 -p 5000:5000 spider_proxy_pool:v1
```

这里使用一个我做爬虫时使用的一个`代理ip池`项目，源码已上传到GitHub

仓库地址：https://github.com/xuefeng365/spider_proxy_pool

项目里已经上传了 Dockerfile文件，可以跟进这个文件 ，直接在`Linux`系统构建即可。

![image-20230215202037617](http://biji.51automate.cn/blogs/img/image-20230215202037617.png)

具体操作步骤如下：

```
# 任意选择下载路径
cd /usr/local/src/
# 克隆GitHub项目
git clone https://github.com/xuefeng365/spider_proxy_pool.git
# 进入项目根目录
cd spider_proxy_pool
# 构建镜像
sudo docker build -t spider_proxy_pool:v1 .
# 查看系统里所有镜像
docker images
# 启动容器：爬取并启动web服务。(ip换成你自己的公网ip即可,这里不对外)
docker run --env DB_CONN=redis://:@ip:6379/0 -p 5000:5000 spider_proxy_pool:v1
# 查看效果 -- 获取ip（只供作者自己使用。容器有可能随时停用），感兴趣可以自己制作
http://www.51automate.cn:5000/get/
```

`sudo docker build -t spider_proxy_pool:v1 .`解释：

我们利用build指令生成镜像，-t 参数指定了最终镜像的名字为`spider_proxy_pool:v1`。

如果注意，会看到 docker build 命令最后有一个 **.** 。 **.** 表示当前目录，这里是指将Dockerfile所在的当前目录作为构建上下文目录。

期间使用以下命令时，遇到权限被拒绝的问题

> git clone https://github.com/xuefeng365/spider_proxy_pool.git

具体解决方法 》 查看系列文章 之 [Linux系统克隆仓库时，权限被拒绝](http://www.51automate.cn/2023/02/15/Linux%E5%85%8B%E9%9A%86%E7%88%AC%E8%99%AB%E4%BB%93%E5%BA%93%E6%97%B6%EF%BC%8C%E6%9D%83%E9%99%90%E8%A2%AB%E6%8B%92%E7%BB%9D/)



## 镜像上传到docker hub

可以直接看这个[把本地docker镜像上传到Docker Hub](https://www.cnblogs.com/suenshuai/p/11652437.html)，亲测好用

1. 去 [https://hub.docker.com](https://hub.docker.com/)注册个账号，注册可能需要外网，不太稳定。

2. docker login  **输入Docker Hub的用户名及密码**

   ```
   docker login
   ```

3. 查看镜像 

   ```
   docker images
   ```

4. 更改镜像名，不改会报错

   docker tag 镜像id docker hub账号名:上传到docker hub上的镜像名

   ```
   docker tag spider_proxy_pool:v1 xuefeng365/spider_proxy_pool:v1
   ```

3. 上传镜像，执行命令

   ```
   docker push xuefeng365/spider_proxy_pool:v1
   ```
   
   

![image-20230221110005650](http://biji.51automate.cn/blogs/img/image-20230221110005650.png)



## 拉取镜像文件，并启动服务

Redis 安装

1. 利用docker安装redis

```
sudo docker pull redis
```

2. 启动redis

```
sudo docker run -d --name redis -p 6379:6379 redis --requirepass
```

3. 拉取镜像

```
docker pull xuefeng365/spider_proxy_pool:v1

docker run --env DB_CONN=redis://:@ip:6379/0 -p 5000:5000 xuefeng365/spider_proxy_pool:v1
```

4. 查看效果(你的公网ip+5000端口，这里是假的) 222.222.222.222:5000

