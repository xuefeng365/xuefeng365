# python安装、pip更新下载、批量生成安装第三方库

### Python下载

Python官网下载：https://www.python.org/

Python国内镜像下载：

1. 淘宝镜像：https://registry.npmmirror.com/binary.html?path=python



### pip 更新

```python
# 更新pip
python -m pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple

# 更新时权限不够
python -m pip install --upgrade pip --user

```



### pip 第三方包下载

```python
pip install pywin32 -i https://pypi.tuna.tsinghua.edu.cn/simple 

# -i https://pypi.tuna.tsinghua.edu.cn/simple 是指国内镜像加速 
```

其他国内镜像源

```
http://pypi.douban.com/simple/ 豆瓣
http://mirrors.aliyun.com/pypi/simple/ 阿里
http://pypi.hustunique.com/simple/ 华中理工大学
http://pypi.sdutlinux.org/simple/ 山东理工大学
http://pypi.mirrors.ustc.edu.cn/simple/ 中国科学技术大学
https://pypi.tuna.tsinghua.edu.cn/simple 清华
```



### 批量生成、安装第三方库

- Python项目中，一般都会有一个 requirements.txt 文件
- 主要是用于记录当前项目下的所有依赖包及其精确的版本号，以方便在一个新环境下更快的进行部署

```py
# 在当前路径下 - 生成 requirement.txt 文件
pip freeze > requirements.txt

# 或者可以指定生成路径如 E:\requirements.txt
pip freeze > E:\requirements.txt

# 批量安装
pip install -r requirements.txt


```

安装速度太慢怎么办？

```
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple


pip install -r ..\..\requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple


```



### 需要重点注意的是

一般项目我们都会创建虚拟环境，我们需要的也只是虚拟环境里的安装包。

所有我们需要利用 `venv/Scripts`（ 虚拟环境）下的 `pip` 生成  `requirement.txt` ，同时最好指定一下 生成的 `requirement.txt`的路径，如果不指定，默认是和虚拟环境中的pip同一目录

```
# cd 进入虚拟环境
cd .\venv\Scripts\
# 不指定 文件目录
pip freeze > requirements.txt
```

![image-20231114123009742](http://biji.51automate.cn/blogs/img/image-20231114123009742.png)



指定生成文件路径（推荐使用相对路径，也可以用绝对路径）

```
# cd 进入虚拟环境
cd .\venv\Scripts\
# 使用相对路径（虚拟环境的 上上级）
pip freeze > ../../requirements.txt
# 使用绝对路径
pip freeze > E:\xf\myFavorite-master\requirement.txt
```



![image-20231114140652782](http://biji.51automate.cn/blogs/img/image-20231114140652782.png)



除了 cd .\venv\Scripts\  ， 也可以利用cmd ，像这样

![1111](http://biji.51automate.cn/blogs/img/1111.gif)

如果你没有使用 虚拟环境里的pip去安装，而是使用 **系统环境变量**里的pip

可以指定 安装第三方包的安装目录

```
# 安装依赖包到指定位置(注意一点 保证你当前位置是有requirements.txt文件)
pip install -r requirements.txt --target=./venv/Lib/site-packages  -i  https://pypi.doubanio.com/simple
```

