---
title: pyside2系列：python代码打包为exe遇到的坑
series: pyside2
tags: [界面,exe]
calegories: [pyside2系列]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---
# python代码打包为exe遇到的坑

现象：在pycharm运行正常，但是使用 `pyinstaller`  打包后 执行exe 或者发给朋友朋友在其它机器上运行就有问题。这些现象都很有可能是由程序使用的文件路径发生改变产生的。



## 报错一

使用以下命令

```
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -F -w C:\Users\Administrator.USER-20190415PP\Desktop\xin
_kefu_ui_qianniu\main.py
```

打包生成的exe文件，打开文件总是报错，报错信息如下

```
Failed to execute script main
```



因为当前exe因为无法看到具体的报错信息，需要重新打包 ，命令如下

```
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -F C:\Users\Administrator.USER-20190415PP\Desktop\xin
_kefu_ui_qianniu\main.py
```



`**-w**` ( 运行exe时就会出现cmd窗口 ) 获取更多信息, 方便排查

![](http://biji.51automate.cn/blogs/img/20230407111114.png)

经过观察：exe文件 默认加载了 PySide2.QtXml 这个文件， 但是该模块不是我代码里写的 ， 打包命令里添加 参数 `--hidden-import PySide2.QtXml` 重新打包即可

```
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -F -w C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\main.py --hidden-import PySide2.QtXml
```



## 报错二

可执行文件无法获取到 关联的项目文件

![image-20230407111905918](http://biji.51automate.cn/blogs/img/image-20230407111905918.png)



查阅资料后发现 :之前代码里写的获取 当前文件路径 `os.path.dirname(__file__) `在pycharm运行没问题， 打包后因为是所有依赖 生成了 一个.exe 文件，执行exe的时候，电脑会生成一个临时缓存，基于缓存获取到的当前文件路径肯定是错误的



解决方案 ： 基于生成的exe 来获取当前路径（不管这个exe 放在哪里 都是基于当前的exe 获取的绝对路径）作为基准路径，如果有其他地方（比如 图标 配置文件等） ，都是根据 基准路径 来定位

```
self.project_path = os.path.dirname(os.path.abspath(sys.executable))

# 开发配置信息
self.con_developer = JsonPars.load(f'{self.project_path}\\qianniu\\cof\\config')
# self.con_developer = JsonPars.load(f'{self.project_path}' + r"\qianniu\cof\config")
print(self.con_developer)

# 用户配置信息路径
self.user_cof_path = f'{self.project_path}\\qianniu\\cof\\peizhi.txt'
# self.user_cof_path = f'{self.project_path}' + r"\qianniu\cof\peizhi.txt"
```



## 报错三：

缺少包：明明已经安装了第三方包， 打开exe提示缺少包。

解决方案：待导入的`.py` 功能模块 最好是和主程序的入口`.py`文件放在**同级目录**



## 报错四：

使用`pyinstaller`打包程序后`chromedriver.exe`弹出了 cmd小黑框，严重影响用户体验。即便我打包命令里 已经加了 `-w` 不显示cmd的参数，还是会出现小黑框。

这里前前后后换了很多方案，比如 原先执行shell 命令 ，是通过 `os.system('cmd命令')`，后面使用 `subprocess.Popen('cmd命令',shell=True)`都不行。

睡了一觉细想，只是执行`chromedriver.exe`才会出现黑框，经过其他指令都不会。肯定出现在`调用chromedriver.exe`这里。

再次查阅资料发现解决方案 如下

找到你安装的selenium库，这里我用的虚拟环境`\venv\Lib\site-packages\selenium\webdriver\common`, 找到 `services.py`, 然后找到`start()`，如下图，添加配置参数 `creationflags=134217728` 即可

![img](http://biji.51automate.cn/blogs/img/20230407102602.png)



## 警告：

如何修复：“在创建QCoreApplication之前必须设置Attribute Qt:：AA-enableHighdDiscaling。”警告

![image-20230407114610734](http://biji.51automate.cn/blogs/img/image-20230407114610734.png)



这可以通过将matplotlib更新到最新版本来解决。首先，使用以下方法删除旧版本：

```
# 升级最新版 ， 如果没有安装 （直接安装最新版）
pip -U install matplotlib
```



## 拓展

```
# exe 无黑框  -F -w
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -F -w C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\main.py --hidden-import PySide2.QtXml

# exe 有黑框 -F
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -F -w C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\main.py --hidden-import PySide2.QtXml

# python环境 无黑框  -D -w
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -D -w C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\main.py --hidden-import PySide2.QtXml

# python环境 有黑框  -D
pyinstaller --icon=C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\qianniu\ico\tubiao1.ico -D C:\Users\Administrator.USER-20190415PP\Desktop\xin_kefu_ui_qianniu\main.py --hidden-import PySide2.QtXml
```

