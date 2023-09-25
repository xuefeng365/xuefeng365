# linux及docker常用命令

⽂件和⽬录操作

```
# 回上级⽬录
cd ..

# 回上⼀步所在⽬录
cd -

# 返回根⽬录
cd

# 显示当前路径
pwd

# 查看⽂件⽬录列表
ls

# 查看⽂件和⽬录的详情列表（增强⽂件⼤⼩易读性）
ls -lh

# 查看⽂件和⽬录的树形结构
tree

# 重命名/移动⽬录
mv old_dir new_dir

# 创建⽬录
mkdir dir1

# 删除文件 （-f：强制删除）
rm -f  test1  

# 删除文件夹+子文件  （-r：递归删除）
rm -rf  test1  

# 删除空文件夹，如果非空，命令不生效
rmdir test1 

# 复制文件
cp file1 file2


# 从跟⽬录查找⽂件/⽬录
find / -name python3.7

# 在⽬录/dir中搜带有.bin后缀的⽂件
find /dir -name *.bin

# 当前⽬录下 （递归）搜带有.bin后缀的⽂件
find . -name *.bin

# 快速查找⽂件（相当于find -name）
locate <关键词>

# 寻找.mp4结尾的⽂件
locate *.mp4

# 显示某⼆进制⽂件/可执⾏⽂件的路径
whereis <关键词>   whereis python3.7

# 查找系统⽬录下某的⼆进制⽂件
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
# 创建指向⽂件/⽬录的软链接
ln -s file1 link1
# 比如 ln -s /var/local/python3/bin/python3.9 /usr/bin/python3

rm -rf python3    #去到创建软链接的目录/usr/bin/下面，使用该命令删除软链，只删除软链接，但不删除实际数据。如果使用rm -rf python3/ 这样删除会把原来 python3 下的内容删除
```



添加软连接和添加到环境变量，好像实现的功能都一样，具体有什么区别？

> 软链接：类似windows中在桌面上建立的一个快捷方式，快捷方式直接指向源文件所在的位置。好处是只是一个链接，占用资源大小忽略不计。




⽂件查看和处理

```
# 查看⽂件内容
cat file1

# 查看⽂件后两⾏
tail -2 file1

# 查看⽂件前两⾏
head -2 file1

# 在⽂件hello.txt中查找关键词codesheep
grep codesheep hello.txt

# 在⽂件hello.txt中查找以sheep开头的内容
grep ^sheep hello.txt

# 查看从第⼀⾏到第5⾏内容
sed -n '1,5p;5q' hello.txt


```

[查找命令 ： grep(文本搜索工具)、find、locate、whereis、which、type 区别](https://www.cnblogs.com/gyrgyr/p/6101750.html)




系统信息和性能查看

```
# 查看linux版本信息
cat /proc/version

# 查看系统环境变量
env

# 查看内存⽤量和交换区⽤量
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

# 过滤出你需要的进程
ps -ef | grep xuefeng365

# kill指定pid的进程
kill -s pid

# kill指定名称的进程
kill -s name

```

⽤户和⽤户组

```
# 创建⽤户
useradd codesheep
# 删除⽤户
userdel -r codesheep
# 修改⽤户的组
usermod -g group_name user_name
# 将⽤户添加到组
usermod -aG group_name user_name

# 创建⽤户组
groupadd group_name
# 删除⽤户组
groupdel group_name
# 重命名⽤户组
groupmod -n new_name old_name

# 查看test⽤户所在的组
groups test

# 查看活动⽤户
w
# 查看指定⽤户xuefeng365信息
id xuefeng365
# 查看⽤户登录⽇志
last

# 查看系统所有组
cut -d: -f1 /etc/group
# 查看系统所有⽤户
cut -d: -f1 /etc/passwd

# 修改⼝令
passwd
# 修改某⽤户的⼝令
passwd codesheep
```



⽂件查看和处理

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

