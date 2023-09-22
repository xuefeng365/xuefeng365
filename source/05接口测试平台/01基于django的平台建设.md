---
title:  基于django的平台建设
---



# 平台建设

1. models.py 文件里 编写不同模型类  

   ![](http://biji.51automate.cn/blogs/img/20221128181133.png)

2. 执行以下命令同步到数据库中，创建表结构

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

   

3. 使用Django的`fixtures`给项目添加初始化数据

   ![](http://biji.51automate.cn/blogs/img/20221128181650.png)

   

   

## 相关知识 

#### 视图

1. 函数视图

2. 类视图

   > 可以继承，提高代码复用

### Django认证系统-Session

![](http://biji.51automate.cn/blogs/img/20221129111359.png)

用户访问服务器，服务器会生成一个session会话（该会话拥有唯一sessionid），sessionid会返回给用户，每次请求带上，服务器跟进id查找会话中的内容

session 会话里包含2方面重要内容

1. 登录后的用户凭证-hash密码
2. 认证后端



https://www.cnblogs.com/liuwchao/articles/12352580.html



### wsgi和asgi

wsgi和asgi都是基于Python设计的 Web服务器 网关接口。
WSGI是基于http协议模式开发的,不支持websocket，而ASGI的诞生解决了python中的WSGI不仅支持当前的web开发中的一些新的协议标准,同时ASGI支持原有模式和Websocket的扩展,即ASGI是WSGI的扩展
网关接口它们是一种规范，描述了Web服务器如何与Web应用程序（客户端）通信，以及如何将Web应用程序链接在一起以处理一个请求。
https://www.cnblogs.com/niuyeji648/p/14867968.html



### Django框架 URLconf模块

https://blog.csdn.net/Richardlcx/article/details/106578515
