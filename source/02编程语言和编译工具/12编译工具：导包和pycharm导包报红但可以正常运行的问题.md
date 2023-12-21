# pycharm相关配置、配置测试用的实时代码模板

##  导包

## pycharm中代码运行没有问题，却标红引用

https://blog.csdn.net/qq_34687559/article/details/122474888

很有可能是 pycharm中python工作区的问题

比如之前我写的一个 前后台分离项目，前后端代码在同一个工程文件里。

但是 正常情况下 开发后端项目会有一个后端工作区，前端项目有一个前端工作区 

![image-20231121115259224](http://biji.51automate.cn/blogs/img/image-20231121115259224.png)

如果pycharm 没有区分`工作区`，就会出现from .. . import 导入模块 报红的问题，因为 以`主项目工程根目录`去定位 `引入路径`肯定是有问题的。

![image-20231121115745187](http://biji.51automate.cn/blogs/img/image-20231121115745187.png)



## pycharm 中的全局搜索(ctrl+shift+f) 功能无法使用
原因是：本机使用了`搜狗输入法`，导致 `全局搜索快捷键` 冲突。
https://blog.csdn.net/lezeqe/article/details/87900438
