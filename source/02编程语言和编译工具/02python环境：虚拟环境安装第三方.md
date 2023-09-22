# python虚拟环境安装第三方包
## python：CMD终端使用pip命令安装包到虚拟环境

已创建虚拟环境，但是pycharm2种方式安装第三方包 `requests_toolbelt`，库里找不到 

![image-20230216112810921](http://biji.51automate.cn/blogs/img/image-20230216112810921.png)



第一时间想到 还可以用pycharm的终端命令 `pip install requests_toolbelt`  安装，提示已经存在

![image-20230129191329521](http://biji.51automate.cn/blogs/img/image-20230129191329521.png)

为什么会提示已经存在呢？原因如下

终端(包括pycharm自带的终端) 直接`pip`安装 ,其实是安装到项目解析器目录下（D:\Python\Python39\Lib\site-packages） ，而非现在项目的虚拟环境里`\venv\Lib\site-packages`。



CMD如何使用pip 安装包到虚拟环境`\venv\Lib\site-packages`呢？

正确的做法，如下

> 进入到venv文件目录下 ，用pip install 指令安装

方法1（任意CMD，指定指定目录参数 `--target`）

```python
pip instal1 requests_toolbelt --target=D:locustl\venv\Lib\site-packages  -i  https://pypi.doubanio.com/simple
```

方法2 （指定目录下`locustl\venv\Lib\site-packages`启动CMD,系统会优先使用当前项目的pip）

```
pip instal1 requests_toolbelt -i  https://pypi.doubanio.com/simple
```



![image-20230129191353404](http://biji.51automate.cn/blogs/img/image-20230129191353404.png)

至此安装完成！

要点解析：

`D:locustl\venv\Lib\site-packages` 是建立的虚拟环境第三方包存放路径，后面的 【 `-i  https://pypi.doubanio.com/simple `  】是指定下载源。



其他国内镜像源, 如下

```
http://pypi.douban.com/simple/ 豆瓣
http://mirrors.aliyun.com/pypi/simple/ 阿里
http://pypi.hustunique.com/simple/ 华中理工大学
http://pypi.sdutlinux.org/simple/ 山东理工大学
http://pypi.mirrors.ustc.edu.cn/simple/ 中国科学技术大学
https://pypi.tuna.tsinghua.edu.cn/simple 清华
```



