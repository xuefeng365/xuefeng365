# python安装、pip更新下载、批量生成安装第三方库

### Python下载

Python官网下载：https://www.python.org/

Python国内镜像下载：

1. 淘宝镜像：https://registry.npmmirror.com/binary.html?path=python



### pip 更新

```python
# 更新pip
python -m pip install --upgrade pip

# 更新时权限不够
python -m pip install --upgrade pip --user

```



### pip 第三方包下载

```python
pip install pywin32 -i https://pypi.tuna.tsinghua.edu.cn/simple 
```

国内镜像加速`-i https://pypi.tuna.tsinghua.edu.cn/simple `

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
```



### 需要重点注意的是

如果要在`虚拟环境`里生成 `requirement.txt` ,需要利用 `venv/Scripts` 下的 `python.exe` 生成  `requirement.txt` 

![](http://biji.51automate.cn/blogs/img/20230130152436.png)

除了使用上述cmd, 也可以用pycharm 终端操作。操作如下

![](http://biji.51automate.cn/blogs/img/20230130152730.png)

