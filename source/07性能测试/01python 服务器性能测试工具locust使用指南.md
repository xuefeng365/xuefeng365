---
title: python服务器性能测试工具locust使用指南
series: locust
tags: [性能,压测,locust]
calegories: [性能]
author: xuefeng365
description: 这里是描述
comments: true
cover: img/1.png
---

# python服务器性能测试工具locust使用指南

## 简介

- 网站：[locust.io](https://gitee.com/link?target=https%3A%2F%2Flocust.io)

- 文档：[docs.locust.io](https://gitee.com/link?target=https%3A%2F%2Fdocs.locust.io)

  

优点：

1. Locust基于Python编写，采用**协程机制**（gevent），它可以避免 **线程和进程的系统资源调度**造成的资源开销，大大**提高单机的并发量**。相比jmeter通过线程启动用户，受服务器本身限制，**单机并发量**，并不高
2. locust工具本身提供一个web平台,可以实时查看数据

缺点：

1. 数据不能存储,所有测试数据存在了内存中,重启以后测试数据会消失
2. 测试结果报表不丰富。

## 安装

```
pip install locust
```

检查是否安装成功

```
locust -V
```

## 基本使用

### 压测需求1

url地址为"https://www.baidu.com/hello" ，get请求，需要模拟100个用户同时访问，每个用户访问url频率在1-5秒之间随机选择，压测10分钟。

```
# demo1.py
import os

from locust import HttpUser, task, between


class QuickstartUser(HttpUser):
    wait_time = between(1, 5)
    host = "https://www.baidu.com"

    @task
    def hello_world(self):
        self.client.get("/hello")
        
if __name__ == '__main__':
    os.system("locust -f demo1.py --host=http://192.168.10.107")
```

本地web启动 --无法定时停止，10分钟后 页面手动停止

```
# 本地web启动 
os.system("locust -f demo1.py --host=http://192.168.10.107")
```

![image-20230203144806971](http://biji.51automate.cn/blogs/img/image-20230203144806971.png)

非web启动  设定执行时间10分钟

```
# 非web启动或Linux服务器使用
os.system("locust -f demo1.py --headless -u 100 -r 2 -t 10m --host=http://192.168.10.107")
```

参数解释：

```
-f demo1.py //代表执行哪一个压测脚本
--headless //代表无界面执行
-u 100 //模拟100个用户操作
-r 2 //每秒用户增长数
-t 10m //压测10分钟, 默认是单位是秒，10m 也可以写成 600
--html report.html //输出的压测结果文件是html，这里也可以指定存放路径
--csv=example  //输出的压测结果文件是excle，这里也可以指定存放路径（用的不多）
-H https://www.baidu.com //定义访问的host(写死)
```

### 压测需求2: 

url地址为"https://www.baidu.com" ，get请求，需要模拟100个用户同时访问，每个用户访问循环访问"/hello"和"/world"这两个接口，任务之间没有间隔（每个用户不停给服务器发送请求，无间隔），并且用户选择这两个接口的比例为1:3，压测10分钟。

```
# demo2.py

import os

from locust import HttpUser, task, between


class QuickstartUser(HttpUser):

    # wait_time = between(1, 5)
    host = "https://www.baidu.com"

    @task(1)
    def hello_world(self):
        self.client.get("/hello")

    @task(3)
    def hello_world(self):
        self.client.get("/world")


if __name__ == '__main__':
    os.system("locust -f demo2.py --headless -u 100 -r 2 -t 10m --html report.html")

```

`@task(3)`、`@task(1)`  定义了任务权重，脚本期望的情况是每执行4次请求，3次是“/world” 1次是"/hello"

**关于任务间隔**

1. 任务随机间隔wait_time = between(a, b) 
2. 任务固定间隔wait_time =constant(a) 
3. 任务无间隔

压测其实就是模拟用户给服务器发请求，**任务无间隔**代表每个用户不停的while 1给服务器发请求，任务固定间隔可以理解成while 1 sleep固定时间给服务器发请求。

如果要求每个用户访问url频率固定2秒，这个怎么办呢？

```
# demo2.py

import os

from locust import HttpUser, task, between


class QuickstartUser(HttpUser):
	# 1-5秒内随机访问
    # wait_time = between(1, 5)
    # 固定访问间隔2秒
    wait_time = constant(2)
    host = "https://www.baidu.com"

    @task(1)
    def hello_world1(self):
        self.client.get("/hello")

    @task(3)
    def hello_world2(self):
        self.client.get("/world")
    

if __name__ == '__main__':
    os.system("locust -f demo2.py --headless -u 100 -r 2 -t 10m --html report.html")

```

**@tag装饰器**

脚本有3个任务，当我们需要选取其中2个任务进行压测时，@tag装饰器就发挥作用了

比如我只想执行 **task2** 、**task3**  ， 执行命令加上`--tags task2 task3`就ok了

```
# demo2.py

import os

from locust import HttpUser, task, between


class QuickstartUser(HttpUser):
	# 1-5秒内随机访问
    # wait_time = between(1, 5)
    # 固定访问间隔2秒
    wait_time = constant(2)
    host = "https://www.baidu.com"

    @task(1)
    @tag("task1")
    def hello_world1(self):
        self.client.get("/hello")

    @task(3)
    @tag("task2")
    def hello_world2(self):
        self.client.get("/world")

    @task(1)
    @tag("task3")
    def hello_world3(self):
        self.client.get("/good")


if __name__ == '__main__':
    os.system("locust -f demo2.py --headless -u 100 -r 2 -t 10m --html report.html --tags task2 task3")

```

**前置与后置**

1. on_start

   执行任务前 先执行比如**登录**或者其他操作

2. on_stop

   任务结束后（当interrupt(中断)或者用户被杀死）执行的操作

on_start方法在模拟用户开始执行该任务集时调用

并且on_stop当模拟用户停止执行该任务集时调用（当interrupt()或者用户被杀死）





## 常用压测场景实战

### 压测需求3: 

**梯度增压**

url地址为"https://www.baidu.com/hello" ，get请求，现在需要对其进行梯度增压，每10分钟增加20个用户，压测3小时，以窥探其性能瓶颈。

```
# demo3.py

import math
import os

from locust import HttpUser, LoadTestShape, task


class QuickstartUser(HttpUser):
    host = "https://www.baidu.com"

    @task
    def hello_world(self):
        self.client.get("hello")


class StepLoadShaper(LoadTestShape):
    '''
    逐步加载实例

    参数解析：
        step_time -- 逐步加载时间
        step_load -- 用户每一步增加的量
        spawn_rate -- 用户在每一步的停止/启动
        time_limit -- 时间限制

    '''
    setp_time = 600
    setp_load = 20
    spawn_rate = 20
    time_limit = 60 * 60 * 3

    def tick(self):
        run_time = self.get_run_time()

        if run_time > self.time_limit:
            return None

        current_step = math.floor(run_time / self.setp_time) + 1

        return (current_step * self.setp_load, self.spawn_rate)


if __name__ == '__main__':
    os.system("locust -f demo3.py --headless -u 10 -r 10 --html report.html")

```



### 压测需求4: 

**保证并发测试数据唯一性，不循环取数据**

url地址为"https://www.baidu.com/hello" ，get请求，有个参数叫id，捞取线上数据进行真实场景压测，保证每个id只请求一次，不循环取数据。

这边使用到了python内置类Queue，他是线程安全的。

```
from queue import Queue

from locust import FastHttpUser, task


def prepare_data():
    q = Queue()
    # 准备ids数据，模拟从文件中读取
    for i in range(100):
        q.put(i)
    return q


class QuickstartUser(FastHttpUser):
    host = "https://www.baidu.com"
    data_queue = prepare_data()

    @task
    def hello_world(self):
        id = self.data_queue.get(timeout=3)
        print(id)
        self.client.get("hello", params={"id": id})
        if self.data_queue.empty():
            print("结束压测")
            exit()

```



### 压测需求5: 

**保证并发测试数据唯一性，循环取数据**

url地址为"https://www.baidu.com/hello" ，get请求，有个参数叫id，捞取线上数据进行真实场景压测，保证每个id只请求一次，循环取数据。

```
from queue import Queue
from locust import FastHttpUser, task


def prepare_data():
    q = Queue()
    # 准备ids数据，模拟从文件中读取
    for i in range(100):
        q.put(i)
    return q


class QuickstartUser(FastHttpUser):
    host = "https://www.baidu.com"
    data_queue = prepare_data()

    @task
    def hello_world(self):
        id = self.data_queue.get()
        print(id)
        self.client.get("hello", params={"id": id})
        self.data_queue.put_nowait(id)

```

这边使用data_queue.put_nowait(id)，id用完之后再塞回队列中



### 压测需求6: 

**非http协议压测**

**websocket协议**

**tcp协议**





本地启动locust服务 

编写服务器脚本 方便在服务器上运行，比如脚本里设置相应参数 无头模式、模拟用户数 、每秒用户增长数、压测时间、压测结果文件输出等

参考教程：https://blog.csdn.net/qq_38783257/article/details/121441097

待总结