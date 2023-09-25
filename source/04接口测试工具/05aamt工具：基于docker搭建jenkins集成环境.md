# aamt工具：基于docker搭建jenkins集成环境

## 一、查看linux 系统是否有docker 

```
docker version
```

 如果没有docker>> CentOS 7下安装Docker及基础操作 

```
一、安装yum

# 1，下载最新的yum-3.2.28.tar.gz并解压
wget http://yum.baseurl.org/download/3.2/yum-3.2.28.tar.gz 
tar xvf yum-3.2.28.tar.gz

# 2，进入目录，运行安装
cd yum-3.2.28 
yum install yum
# 如果结果提示错误： CRITICAL:yum.cli:Config Error: Error accessing file for config file:///etc/
# 可能是原来是缺少配置文件。在etc目录下面新建yum.conf文件，然后再次运行 yummain.py install yum，顺利完成安装。

# 3，最后更新系统。
yum check-update
yum update
yum clean all

# 二、安装docker
# 1.Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker ，通过 uname -r 命令查看你当前的内核版本：
uname -r

# 2.安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 3.设置yum源
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 4.安装docker
sudo yum install docker-ce

# 5.启动并加入开机启动
sudo systemctl start docker
sudo systemctl enable docker

# 6.验证安装是否成功(有client和service两部分表示docker安装启动都成功了)
docker version
```



## 二、Docker 安装Jenkins 



1、查看本地所有 镜像 （非root用户查看使用命令：sudo docker images） 

```
docker images
```



2、docker下载Jenkins镜像 

```
docker pull jenkins/jenkins:2.306-centos


# 删除不用的镜像(先查看，再删除)
# 查看所有的镜像
docker images

# 删除没用的镜像 节省空间
docker rmi 05d7defdb214

# 展示出所有的容器
docker pa -a 

# 删除单个容器（docker rm -f -v 容器名）
docker rm -f -v jenkins2
```



3、新建jenkins的挂载目录 

```
# 在主机下创建一个文件夹（-p 要加上） 用于挂载目录
mkdir -p /home/xuefeng/jenkins_node

# 给挂载目录一个最高权限 （可读可写可执行）
chmod -R 777 /home/xuefeng/jenkins_node
```



4 、创建与启动 jenkins 容器  

```
docker run -d -uroot -p 10086:8080 --name jenkins1 -v /home/xuefeng/jenkins_node:/var/jenkins_home jenkins/jenkins:2.306-centos
```

`-d`： 守护模式

`-uroot`： 使用 root 身份进入容器，推荐加上，避免容器内执行某些命令时报权限错误

`-p`：10086:8080 jenkins的web访问端口10086（通过访问外部端口10086，实际映射到Jenkins容器的 8080端口）

`-v`：目录映射  /home/xuefeng/jenkins_node:/var/jenkins_home 容器/var/jenkins_home路径映射到宿主机/home/xuefeng/jenkins_node

`--name`：自定义一个容器名称  jenkins1

`jenkins/jenkins:2.306-centos` 使用这个镜像

5、查看Jenkins镜像是否启动成功 

```
docker ps -a
```

6、开启docker交互模式终端-进入容器 

```
docker exec -it -uroot jenkins1 bash

# 退出docker 容器
# ctrl + p + q
```

7、访问Jenkins

[http://www.automate.cn:10086](http://www.automate.cn:10086/) 如不能访问就是容器关闭了

8、首次登陆查看密码 



![image (3)](http://biji.51automate.cn/blogs/img/image%20(3).png)

解锁jenkins  -- 容器内  进入目录 `/var/jenkins_home/secrets/initialAdminPassword` 查看文件`initialAdminPassword`

```
cat /var/jenkins_home/secrets/initialAdminPassword
```



9、更改插件源地址，提升下载插件的速度 

[http://www.automate.cn:10086/pluginManager/advanced](http://www.automate.cn:10086/)，进入插件源配置界面，滑到最后，设置插件源地址： 

[https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fjenkins%2Fupdates%2Fupdate-center.json) 设置好后点击保存，最后点击检查。

![image (2)](http://biji.51automate.cn/blogs/img/image%20(2).png)



![image (1)](http://biji.51automate.cn/blogs/img/image%20(1).png)

如果速度还是慢 ，[点这里找答案](https://cloud.tencent.com/developer/article/1547673)



## 三、前置准备（重要） 

 1.**获取最新的软件包**  

```
yum update
```

 2.**提前安装，以便接下来的配置操作**  

```
yum -y install gcc automake autoconf libtool make 
yum -y install make* 
yum -y install zlib* 
yum install openssl-devel -y // 
yum -y install sudo
```



## 四、容器内安装python3环境（在线安装） 

 1.进入jenkins_home目录 

```
cd /var/jenkins_home/ 
```

 2.创建python存放路径  

```
mkdir python3 

cd python3
```

 3.下载python3 使用wget时发现没有该命令  

```
# 安装 wget（yum 会自动下载wget 所需依赖包）
yum -y install wget

wget http://npm.taobao.org/mirrors/python/3.9.0/Python-3.9.0.tgz
# 默认下载到当前文件（这里是python3文件夹）
```

 4.解压文件目录  

```
tar -zxvf Python-3.9.0.tgz 
```

 5.查看解压后的文件  

```
ls 
```



## 五、make编译安装 

在Python3.9.0目录下

 **第一步**  

```
cd Python-3.9.0
# 配置编译路径
./configure --prefix=/var/jenkins_home/python3 --with-ssl 
```

---

遇到的问题：执行./configure时报错：

```
configure: error: no acceptable C compiler found in $PATH
```

查看得知未安装合适的编译器，可以使用下面的命令安装编译器，解决./configure时报错。

```
# 使用以下命令 会自动安装/升级gcc及其他依赖的包
sudo yum install gcc-c++  
```

重新配置编译路径即可

```
./configure --prefix=/var/jenkins_home/python3 --with-ssl 
```

 

**第二步**  

```
# 编译
make && make install 
```



## 六、添加软链 

 添加python3软链接 

```
ln -s /var/jenkins_home/python3/bin/python3.9 /usr/bin/python3
```

 添加pip3软链接 

```
ln -s /var/jenkins_home/python3/bin/pip3 /usr/bin/pip3

# 如果之前添加过，会提示文件存在 ln:failed to create symbolic link '/usr/bin/pip3': File exists，解决方案：覆盖文件，执行以下命令
ln -sf /var/jenkins_home/python3/bin/pip3 /usr/bin/pip3

```

除了添加软连，还可以直接 把 python3 pip3 添加到环境变量



验证是否成功 

```
pip3
python3
```

![image-20230216100517886](http://biji.51automate.cn/blogs/img/image-20230216100517886.png)



## 七、安装allure 

 1、将allure压缩包传送到容器内（分2步） 

- 先将包上传到主机  /usr/local/src/  目录下
- 然后从主机复制到容器 /var/jenkins_home/ 目录下

```
# 退出容器： 
ctrl + p + q

# 复制文件到容器
docker cp /usr/local/src/allure-2.15.0.zip jenkins1:/var/jenkins_home/ 

# 重新进入容器
docker exec -it -uroot jenkins1 bash

# cd 到容器的指定目录
cd /var/jenkins_home/

# 解压
unzip allure-2.15.0.zip

# 重命名
mv allure-2.15.0 allure

# 赋予文件夹所有内容最高权限
chmod -R 777 allure
```



 2、添加allure软链接 

```
ln -s /var/jenkins_home/allure/bin/allure /usr/bin/allure 
```



 3、配置allure 和 py 环境变量 

 [Linux环境变量配置全攻略](https://www.cnblogs.com/youyoui/p/10680329.html)

```
# 记得一行一个回车哦，不然就直接复制粘贴

cat >> /root/.bashrc << "EOF" 
export PATH=/home/data/allure/bin:$PATH 
export PATH=/var/jenkins_home/python3/bin:$PATH 
EOF
```

 拓展：查看所有的环境变量

```
export
```



4、更新环境变量配置文件 

```
source /root/.bashrc
```



 5、验证环境变量 

```
allure --version 
python3 --version
```



## 八、安装JDK（版本1.8以上） 



查看当前是否有jdk的版本：

```
java -version 
```



查看到该容器已经有了 1.8.0_292, 直接配置

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25763438/1641792584841-3d1bf03f-9c0a-4d66-8c9c-82f2bc326559.png)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25763438/1641793632110-a9cd5f6b-b580-40a5-a9e9-f2a1c1fda71c.png)







### 如果没有JDK环境，安装以下方式安装

 1、将JDK压缩包传送到容器内（分3步） 

- 下载jdk：华为镜像：https://repo.huaweicloud.com/java/jdk/

- 将包上传到主机  `/usr/local/src/`  目录下

- 然后从主机复制到容器 `/var/jenkins_home/` 目录下

  ![image.png](http://biji.51automate.cn/blogs/img/1641658434019-53f9fbcb-6c6e-4808-90b3-f42d64755f90.png)

```
# 退出容器： 
ctrl + p + q

# 复制文件到容器
docker cp /usr/local/src/jdk-8u301-linux-x64.tar.gz jenkins1:/var/jenkins_home/
```



 2、解压  

```
# 重新进入容器
docker exec -it -uroot jenkins1 bash

# cd 到容器的指定目录
cd /var/jenkins_home/

# 解压
tar -zxvf jdk-8u301-linux-x64.tar.gz

# 查看文件
ls
```



 3、重命名 

```
mv jdk1.8.0_301 jdk
```



 4、赋予文件夹所有内容最高权限 

```
chmod -R 777 jdk
```



 5、将JDK路径加入环境变量中 

这里只展示一种方法，更多方法 看这里： [Linux环境变量配置全攻略](https://www.cnblogs.com/youyoui/p/10680329.html)

```
vi /etc/profile

export JAVA_HOME=/var/jenkins_home/jdk/    # 这里换成你的JDK路径
export PATH=$JAVA_HOME/bin:$PATH

```

![image.png](http://biji.51automate.cn/blogs/img/1641660861625-ac5dafca-ee0f-4e90-8b99-fdb5ecfb1ec4.png)



- 按 i 进入插入模式

- 末尾输入下面的代码

- 按`ESC`退出插入模式

- 输入 `:wq` 保存退出



同理这里也可以把python3 、pip3、或者第三方pytest库 添加到环境变量 ,具体可以找时间看看。



 6、让环境变量立即生效 

```
source /etc/profile
```



 7、验证是否成功  （注意是 `-version` 是一个横线） 

```
java -version 
```

![image-20230216102356899](http://biji.51automate.cn/blogs/img/image-20230216102356899.png)




## 九、安装项目需要的第三方库 



1、通过  requirement.txt 批量安装

在 python 项目生成一个 requirement.txt (如果项目里有虚拟环境，生成的时候，需要注意。)

[python系列文章: python安装、pip更新下载、批量生成安装第三方库](http://www.51automate.cn/2023/01/30/python%E5%AE%89%E8%A3%85%E3%80%81pip%E6%9B%B4%E6%96%B0%E4%B8%8B%E8%BD%BD%E3%80%81%E6%89%B9%E9%87%8F%E7%94%9F%E6%88%90%E5%AE%89%E8%A3%85%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93/)

给当前项目分配一个单独的：python解析器，只生成当前项目依赖的第三方库

```
解决办法：把超时的单个包拎出来
pip3 —default-timeout=1000 install -U 包名
比如pip3 —default-timeout=1000 install -U Bebel==2.7.0
然后就慢慢等吧....搞定
```

将 requirement.txt 上传到主机

```
docker cp /usr/local/src/requirements.txt jenkins1:/var/jenkins_home/python3
```



重新进入容器

```
docker exec -it -uroot jenkins1 bash
```



官方仓库安装

```
# 方式一（缺点一旦有一个出错，pip3 list 一下，发现第三方库 全部没安装成功）
pip3 install -r requirements.txt

# 方式二（解决方式一出现的问题）
for i in `cat requirements.txt `;do pip3 install $i;done

```



安装速度太慢怎么办？

```
# 方式一（缺点一旦有一个出错，pip3 list 一下，发现第三方库 全部没安装成功）
pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 方式二（解决上面出现的问题）
for i in `cat requirements.txt `;do pip3 install $i -i https://pypi.tuna.tsinghua.edu.cn/simple;done
```



更多国内镜像源：

```
http://pypi.douban.com/simple/ 豆瓣
http://mirrors.aliyun.com/pypi/simple/ 阿里
http://pypi.hustunique.com/simple/ 华中理工大学
http://pypi.sdutlinux.org/simple/ 山东理工大学
http://pypi.mirrors.ustc.edu.cn/simple/ 中国科学技术大学
https://pypi.tuna.tsinghua.edu.cn/simple 清华

```



2、如果批量安装出错

![image.png](http://biji.51automate.cn/blogs/img/1641830513988-2675f92e-7eba-46d1-a438-830f75feb496.png)



可以尝试单个安装

```
pip3 install jsonpath
```



![image.png](http://biji.51automate.cn/blogs/img/1641830569180-b7af8362-b0ed-4b2a-a339-ac7bb4f8db59.png)



单个安装还出错，就只能手工下载，传到容器内部 ， 再安装

官网下载 https://pypi.org/project/

![image.png](http://biji.51automate.cn/blogs/img/1641830753548-537d032e-1486-4213-904a-f0a4d0304fa3.png)



![image.png](http://biji.51automate.cn/blogs/img/1641831896909-6b2472de-4466-48ea-83f9-cb5d0725789f.png)



```
# 复制到容器里
docker cp /usr/local/src/jsonpath-0.82.tar.gz jenkins1:/var/jenkins_home/

# 重新进入容器
docker exec -it -uroot jenkins1 bash

cd /var/jenkins_home/
# 解压
tar -zxvf jsonpath-0.82.tar.gz
# 进入jsonpath-0.82

cd jsonpath-0.82
# 构建
python3 setup.py build
# 安装
python3 setup.py install 
# 验证 
pip3 list
```



![image.png](http://biji.51automate.cn/blogs/img/1641831571820-81c657c5-3fd8-4fd0-bb65-994f96334b34.png)



## 10 查看效果 

http://www.51automate.cn:10086/



Jenkins部署好之后，如何创建、配置项目仓库、构建规则、邮件通知等？ 

请查看 aamt 系列文章>>[ Jenkins创建、配置项目仓库、构建规则、邮件通知](http://www.51automate.cn/2023/02/16/aamt%E5%B7%A5%E5%85%B7%EF%BC%9AJenkins%E5%88%9B%E5%BB%BA%E3%80%81%E9%85%8D%E7%BD%AE%E9%A1%B9%E7%9B%AE%E4%BB%93%E5%BA%93%E3%80%81%E6%9E%84%E5%BB%BA%E8%A7%84%E5%88%99%E3%80%81%E9%82%AE%E4%BB%B6%E9%80%9A%E7%9F%A5/)





## 扩展

成功构建 jenkins构建python自动化项目时，控制台提示：执行shell报错 `pytest:command not found `

https://blog.csdn.net/csdn_wuxinzy/article/details/119213802

明明已经安装过了

![image.png](http://biji.51automate.cn/blogs/img/1676377214342-c213a15c-68b3-401d-897d-9564a2deeac8.png)



原因就是执行目录找不到pytest命令， 可使用pytest的安装目录，也可将安装目录添加软链接到 系统的环境变量目录。

不管是Linux 还是 windows 执行命令时，都是先从当前目录查找命令， 如果没有此命令， 就会去环境变量的目录去查找命令。

使用 以下命令 当前所有环境变量列表。 [Linux export 命令 拓展>> 菜鸟教程](https://www.runoob.com/linux/linux-comm-export.html)

```
export
```

![image-20230216182016358](http://biji.51automate.cn/blogs/img/image-20230216182016358.png)



```
# 容器内先查找到pytest的路径
find / -name pytest
```



- 解决方案1, 将其添加到软连接（推荐）

![image.png](http://biji.51automate.cn/blogs/img/1676376286821-275e8d7a-4c70-4e90-991a-10191a4fb449.png)

- 解决方案2,  使用 pytest的绝对路径 `/var/jenkins_home/python3/bin/pytest  -v -s  `  执行pytest命令



