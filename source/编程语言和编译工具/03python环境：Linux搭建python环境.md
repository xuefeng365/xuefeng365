# python环境：Linux搭建python环境

### 卸载

1. 卸载python3

   ```
   rpm -qa|grep python3|xargs rpm -ev --allmatches --nodeps 卸载pyhton3
   ```

2. 删除所有残余文件

   ```
   whereis python3 |xargs rm -frv
   ```

3. 查看现有安装的python

   ```
   whereis python
   ```

### 安装python38、python39

```
# 安装python3.9.0(默认)
cd /var/local/
# 安装目录
mkdir python39

cd python39
# wget 在线下载工具（yum 会自动下载wget 所需依赖包）
yum -y install wget

# 下载python3.9
wget http://npm.taobao.org/mirrors/python/3.9.0/Python-3.9.0.tgz

# 解压
tar -zxvf Python-3.9.0.tgz 
# 查看当前目录文件
ls 
# 进入解压目录
cd Python-3.9.0

# 提前准备好 安装和编译python的环境  --prefix 是指 python3的安装路径
./configure --prefix=/var/local/python39 --with-ssl 

# 如果报错 可以执行下面这行命令。sudo指临时获取 超级用户权限。
sudo yum install gcc-c++

# 编译
make && make install 
# 添加软连接
ln -s /var/local/python39/bin/python3.9 /usr/bin/python39
ln -s /var/local/python39/bin/pip3 /usr/bin/pip39

---------------------------------------------------------------
# 安装python3.8.9
cd /var/local/
# 安装目录
mkdir python38

cd python3
# wget 在线下载工具
yum -y install wget
# 下载python3.8.9
wget http://npm.taobao.org/mirrors/python/3.8.9/Python-3.8.9.tgz
# 解压
tar -zxvf Python-3.8.9.tgz 
# 查看当前目录文件
ls 
# 进入解压目录
cd Python-3.8.9
# 提前准备好 安装和编译python的环境  --prefix 是指 python3的安装路径
./configure --prefix=/var/local/python38 --with-ssl 
# 如果报错 可以执行下面这行命令。sudo指临时获取 超级用户权限。
sudo yum install gcc-c++
# 编译
make && make install 
# 添加软连接
ln -s /var/local/python38/bin/python3.8 /usr/bin/python38
ln -s /var/local/python38/bin/pip3 /usr/bin/pip38

# 后期只需要使用 python38即可进入python38版本环境
```

### Docker内安装python环境

无非是增加了 容器的操作：比如 下载镜像，基于镜像创建启动一个容器，然后进入容器，安装python环境（安装过程参考上面）

完整教程可以参考我以下教程，Jenkins容器内安装pyhton环境

https://www.yuque.com/xuefeng365/dftrox/fk9cnn?singleDoc# 《步骤：基于docker搭建jenkins集成环境》

### 项目工程安装python虚拟环境

##### 安装  `virtualenv `工具包，并创建虚拟环境 并安装依赖

```
安装 virtualenv (这里使用的是 pip38 ,因为上面我添加的软连接就是pip38,可能有多个pyhton版本，方便区分)
pip38 install virtualenv -i https://pypi.tuna.tsinghua.edu.cn/simple/

cd /root/fv2/app   # 进入到后端程序代码的根目录

python38 -m virtualenv venv     # 创建虚拟环境

source ./venv/bin/activate      # 进入虚拟环境

pip38 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/   # 安装库
```

