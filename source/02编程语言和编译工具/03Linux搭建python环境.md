# Linux搭建python环境

## 超实用卸载、安装python3

###  卸载

1. 卸载python3

   ```
   rpm -qa|grep python3|xargs rpm -ev --allmatches --nodeps 卸载pyhton3
   ```

2.  删除所有残余文件

   ```
   whereis python3 |xargs rm -frv
   ```

3. 查看现有安装的python

   ```
   whereis python
   ```

### 安装

```

cd /var/local/
# 安装目录
mkdir python3

cd python3
# wget 在线下载工具
yum -y install wget
# 下载python3.9
wget http://npm.taobao.org/mirrors/python/3.9.0/Python-3.9.0.tgz
# 解压
tar -zxvf Python-3.9.0.tgz 
# 查看当前目录文件
ls 
# 进入解压目录
cd Python-3.9.0
# 
./configure --prefix=/var/local/python3 --with-ssl 
# 如果报错 可以执行下面这行命令
sudo yum install gcc-c++
# 编译
make && make install 
# 添加软连接
ln -s /var/local/python3/bin/python3.9 /usr/bin/python3
ln -s /var/local/python3/bin/pip3 /usr/bin/pip3
```

