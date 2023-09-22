---
title: aamt工具：pytest运行控制台中文乱码的问题解决方案
series: 接口自动化
tags: [aamt,测试,pytest]
calegories: [接口自动化]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---



# 关于pytest运行完成后，控制台中文乱码的问题

我在运行pytest的时候，在控制台输出以后呢，会遇见一些中文乱码的问题

![image-20230109114858040](http://biji.51automate.cn/blogs/img/image-20230109114858040.png)



解决方案：在`conftest.py`文件中加上以下代码就可以了 (该代码已经封装到`aamt`接口自动化工具里)

```python
# conftest.py

def pytest_collection_modifyitems(items):
    """
    测试用例收集完成时，将收集到的item的name和nodeid的中文显示
    :return:
    """
    for item in items:
        item.name = item.name.encode("utf-8").decode("unicode_escape")
        item._nodeid = item.nodeid.encode("utf-8").decode("unicode_escape")

```

再次执行用例，控制台如下

![image-20230109115010866](http://biji.51automate.cn/blogs/img/image-20230109115010866.png)

