# linux及docker常用命令

文件和目录操作

```
# 回上级目录
cd ..

# 回上一步所在目录
cd -

# 返回根目录
cd

# 显示当前路径
pwd

# 查看文件目录列表
ls

# 查看文件和目录的详情列表（增强文件易读性）
ls -lh

# 查看文件和目录的树形结构
tree

# 重命名/移动目录
mv old_dir new_dir

# 创建目录
mkdir dir1

# 删除文件 （-f：强制删除）
rm -f  test1  

# 删除文件夹+子文件  （-r：递归删除）
rm -rf  test1  

# 删除空文件夹，如果非空，命令不生效
rmdir test1 

# 复制文件
cp file1 file2


# 从跟目录查找文件/目录
find / -name python3.7

# 在目录/dir中搜带有.bin后缀的文件
find /dir -name *.bin

# 当前目录下 （递归）搜带有.bin后缀的文件
find . -name *.bin

# 快速查找文件（相当于find -name）
locate <关键词>

# 寻找.mp4结尾的文件
locate *.mp4

# 显示某二进制文件/可执行文件的路径
whereis <关键词>   whereis python3.7

# 查找系统目录下某的二进制文件
which <关键词>  which python3.7


```



[env/export命令的区别](https://blog.csdn.net/Scirhh/article/details/116941807)

> 1. 都是显示环境变量
> 2. export输出内容是：按变量名进行排序后的

$PATH 环境变量： Linux 系统比较常用的变量之一。格式：多个路径组成，由英文冒号(:)进行分割

![image-20230221161320756](http://biji.51automate.cn/blogs/img/image-20230221161320756.png)

假设将用户家目录下的 tools 添加 $PATH环境变量，怎么操作？

```
export PATH=$PATH:/home/ascmcs/tools
```



```
# 创建指向文件/目录的软链接
ln -s file1 link1
# 比如 ln -s /var/local/python3/bin/python3.9 /usr/bin/python3

rm -rf python3    #去到创建软链接的目录/usr/bin/下面，使用该命令删除软链，只删除软链接，但不删除实际数据。如果使用rm -rf python3/ 这样删除会把原来 python3 下的内容删除
```



添加软连接和添加到环境变量，好像实现的功能都一样，具体有什么区别？

> 软链接：类似windows中在桌面上建立的一个快捷方式，快捷方式直接指向源文件所在的位置。好处是只是一个链接，占用资源大小忽略不计。




文件查看和处理

```
# 查看文件内容
cat file1

# 查看文件后两行
tail -2 file1

# 查看文件前两行
head -2 file1

# 在文件hello.txt中查找关键词codesheep
grep codesheep hello.txt

# 在文件hello.txt中查找以sheep开头的内容
grep ^sheep hello.txt

# 查看从第1行到第5行内容
sed -n '1,5p;5q' hello.txt


```

[查找命令 ： grep(文本搜索工具)、find、locate、whereis、which、type 区别](https://www.cnblogs.com/gyrgyr/p/6101750.html)




系统信息和性能查看

```
# 查看linux版本信息
cat /proc/version

# 查看系统环境变量
env

# 查看系统的内存使用情况
free -h

# Linux手动清除缓存的方法（少用）
sync&&echo 3 > /proc/sys/vm/drop_caches

```

![image-20230214180755293](http://biji.51automate.cn/blogs/img/image-20230214180755293.png)

> total: 内存总数
>
> used: 已经使用内存数
>
> free: 完全空闲内存
>
> shared: 多个进程共享的内存
>
> buffers: 用于块设备数据缓冲，记录文件系统metadata（目录，权限，属性等)
>
> cached: 用于文件内容的缓冲
>
> available：真正剩余的可被程序应用的内存数



docker容器

```
# 安装查看版本 
docker -v 

# 查看当前系统中的所有镜像
docker images  

# 查看系统中正在运行的容器
docker ps 

# 查看所有容器（包括已停止的容器）
docker ps -a   

# 删除单个容器
docker rm -f -v 容器名   


# 启动容器
docker start  
# 重启容器
docker restart  
# 停止容器
docker stop 

# 重命名容器(docker rename jenkins3 jenkins1)
docker rename old new 

docker run：启动某个镜像，运行一个容器
# docker run -d -uroot -p 10086:8080 --name jenkins1 -v /home/xuefeng/jenkins_node:/var/jenkins_home jenkins/jenkins:2.306-centos

docker run --env DB_CONN=redis://:@ip:6379/0 -p 5000:5000 xuefeng365/spider_proxy_pool:v1


# 进入容器 
docker exec -it jenkins1 bash  
# 退出容器
exit # 或者键盘按 Ctrl + P + Q 也可退出


docker pull ：从docker hub拉取某个镜像

docker build：根据Dockerfile创建一个镜像

```



[扩展：>> docker run详解](https://www.runoob.com/docker/docker-run-command.html)



打包解压

```
# 解压文件 
tar  Python-3.7.0.tgz
```



进程管理

```
# 查看所有进程
ps -ef

# 实时显示进程状态
top

# ps -ef 所有进程; grep 用来过滤包含"xuefeng365"的进程
ps -ef | grep xuefeng365

# kill指定pid的进程
kill -s pid

# kill指定名称的进程
kill -s name


# 查找端口8888被哪些程序占用
lsof -i:8888  # 会显示占用的程序名称和进程ID
# netstat -ap 显示所有网络连接和监听端口; grep 用来过滤包含"8888"的行
netstat -ap|grep 8888
```



用户和用户组

```
# 创建用户
useradd codesheep
# 删除用户
userdel -r codesheep
# 修改用户的组
usermod -g group_name user_name
# 将用户添加到组
usermod -aG group_name user_name

# 创建用户组
groupadd group_name
# 删除用户组
groupdel group_name
# 重命名用户组
groupmod -n new_name old_name

# 查看test用户所在的组
groups test

# 查看活动用户
w
# 查看指定用户xuefeng365信息
id xuefeng365
# 查看用户登录日志
last

# 查看系统所有组
cut -d: -f1 /etc/group
# 查看系统所有用户
cut -d: -f1 /etc/passwd

# 更改用户的密码
passwd
# 修改某用户的密码
passwd codesheep
```



文件查看和处理

```

```

RPM包管理命令

```

```

YUM包管理命令

```
```

DPKG包管理命令

```
```

APT软件⼯具

```

```



清理日志

Linux /var/log/日志文件太大，清理journal就行

```
# 系统根目录
cd /
# 使用命令检查磁盘文件 查看日志使用量
df -h
或者
du -sh *|sort -h
或者 
du -h --max-depth=1

cd进入查看比较大的文件
再次du -sh *|sort -h


```

![image-20231009193923458](http://biji.51automate.cn/blogs/imgimage-20231009193923458.png)

![image-20231009194634435](http://biji.51automate.cn/blogs/imgimage-20231009194634435.png)



亲测有效：docker清理Overlay2占用磁盘空间

https://blog.csdn.net/Small_StarOne/article/details/123655176



https://blog.51cto.com/u_16213367/7649711#:~:text=%E6%B8%85%E7%90%86Overlay2%20%E6%AD%A5%E9%AA%A41%3A%20%E5%88%97%E5%87%BA%E6%89%80%E6%9C%89%E9%95%9C%E5%83%8F%E5%92%8C%E5%AE%B9%E5%99%A8%20%E6%AD%A5%E9%AA%A42%3A%20%E5%88%A0%E9%99%A4%E6%89%80%E6%9C%89%E5%81%9C%E6%AD%A2%E7%9A%84%E5%AE%B9%E5%99%A8,%E6%AD%A5%E9%AA%A43%3A%20%E5%88%A0%E9%99%A4%E6%89%80%E6%9C%89%E6%9C%AA%E8%A2%AB%E4%BD%BF%E7%94%A8%E7%9A%84%E9%95%9C%E5%83%8F%20%E6%AD%A5%E9%AA%A44%3A%20%E6%B8%85%E7%90%86Overlay2%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%20%E6%AD%A5%E9%AA%A45%3A%20%E6%A3%80%E6%9F%A5%E7%A3%81%E7%9B%98%E7%A9%BA%E9%97%B4

有一天使用，突然发现我的 linux服务器 空间占用太高，没空间了。

因为有使用 docker ，猜想长时间运行日志文件肯定很多。下面验证一下 ，发现`-json.log`文件果然很大，删除时即可

```
# 使用命令检查磁盘文件 看看哪些文件过大
df -h
```

![image-20231010095707757](http://biji.51automate.cn/blogs/imgimage-20231010095707757.png)

再次查看占用空间，释放了很多。

![image-20231010095959882](http://biji.51automate.cn/blogs/imgimage-20231010095959882.png)

前面发现还有一个文件`overlay2` 占用空间也很多,我们也清理一下

![image-20231010100350060](http://biji.51automate.cn/blogs/imgimage-20231010100350060.png)

同样的方法，一步步找出 大文件，删除即可，不再记录

