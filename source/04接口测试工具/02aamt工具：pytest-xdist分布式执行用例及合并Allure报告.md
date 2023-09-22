---
title: aamt工具：pytest-xdist分布式执行用例及合并Allure报告
series: 接口自动化
tags: [aamt,测试,pytest]
calegories: [接口自动化]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---

# pytest-xdist分布式执行用例及合并Allure报告

## 分布式执行用例

借助于pytest-xdist，在命令行执行时添加参数`-n auto`：

```
pytest -n auto
```

pytest-xdist会自动根据本地机器硬件配置，设置最优并发，并分发用例，分布式执行。

当然 也可以自主设置进程数量 ，比如 

```
pytest -n 3
# 或者 
pytest -n 4
```

测试方法：

1. 运行框架里面的 `main.py`

![image-20230202184012921](http://biji.51automate.cn/blogs/img/image-20230202184012921.png)

2. 设定用例的采集路径。这里只采集 `main.py` 生成的用例文件

   ![image-20230202184352430](http://biji.51automate.cn/blogs/img/image-20230202184352430.png)

执行用例,

1. 第一次串行,执行以下命令

   ``` pytest
   pytest --aamt-reports
   ```

2. 第二次xdist并行,执行以下命令

   ```
   pytest -n auto --aamt-reports
   ```

![image-20230202185346511](http://biji.51automate.cn/blogs/img/image-20230202185346511.png)

执行时间从50s一下降到4s，性能提升还是非常的明显

## 合并Allure报告

pytest-xdist分布式执行，只要把allure源文件，也就是那一堆json文件，存到同一个目录下，报告的数据就是一体的，不需要单独合并。但是有个问题，aamt封装了`--aamt-reports` 命令行参数一键生成Allure报告，背后的逻辑是在pytest_sessionfinish hook函数里面实现的，分布式执行时，每个xdist的node都是一个单独的session，多个node就会生成多份报告：

![image-20230202185914378](http://biji.51automate.cn/blogs/img/image-20230202185914378.png)

10个node（节点）生成了11份报告，其中1份master节点生成的。pytest-xdist的原理如图所示：

![](http://biji.51automate.cn/blogs/img/20230202185949.png)



master节点不运行任何测试，只是通过一小部分消息与节点通信。子节点执行后会通过workeroutput把数据回传给master节点。所以只需要通过是否有workeroutput属性来判断master节点：

```
def _is_master(config):
    """
    pytest-xdist分布式执行时，判断是主节点master还是子节点
    主节点没有workerinput属性
    """
    return not hasattr(config, 'workerinput')
```

然后只在主节点的`pytest_sessionfinish`，生成1次报告，就能避免生成多份报告。这样在xdist分布式执行模式下，`--aamp-reports`也只会生成1份合并后的包含所有测试用例的Allure HTML报告。

源码实现：

```
```

感谢刚哥，给的建议，

为了能够持续集成，做了这里做了一点点优化。



实际使用过程中， 分布式环境下执行用例，如何有效解决token复用问题？ 会有专门介绍
