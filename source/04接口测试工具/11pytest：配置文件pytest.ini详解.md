---
title: aamt工具：pytest.ini配置文件详解
series: 接口自动化
tags: [aamt,测试,pytest]
calegories: [接口自动化]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---



## 使用pytest --help指令可以查看pytest.ini的设置选项

```
pytest -h
```

结果：

```python
[pytest] ini-options in the first pytest.ini|tox.ini|setup.cfg file found:

  markers (linelist):   markers for test functions
  empty_parameter_set_mark (string):
                        default marker for empty parametersets
  norecursedirs (args): directory patterns to avoid for recursion
  testpaths (args):     directories to search for tests when no files or directories are   given in the command line.
  usefixtures (args):   list of default fixtures to be used with this project
  log_file (string):    default value for --log-file
  log_file_level (string): default value for --log-file-level
  log_file_format (string): default value for --log-file-format
  log_file_date_format (string): default value for --log-file-date-format
  log_auto_indent (string): default value for --log-auto-indent
  faulthandler_timeout (string): Dump the traceback of all threads if a test takes more than         TIMEOUT seconds to finish. Not available on Windows.
  addopts (args):       extra command line options
  minversion (string):  minimally required pytest version
```

## addopts设置项

addopts设置项，可以添加指定的 `OPTS` 命令行参数集，在执行命令时，不用再每次都输入长串命令，就好像它们是由用户指定的一样。示例：如果您有这个ini文件内容：

```
#pytest.ini

[pytest]
;# 选中指定标记smoke执行
;addopts = -v -s -m smoke

;# 同时选中多个标记：只要标记'smoke or test'都执行
;addopts = -v -s -m 'smoke or test'

testpaths = ./case
python_files = test_offer_doctor.py
;python_files = test_offer_nurse.py
;python_files = test_*.py
;python_classes = Test*
;python_functions = test_*

markers =
    smoke:冒烟测试用例
    offer_serve: offer微服务下的接口
    single_api: 单接口参数校验用例
    p0: level p0 (优先级)
    p1: level p1 (优先级)

```

配置好后，执行`pytest`命令或者

执行`main.py`时，

```
# main.py
# -*- coding: UTF-8 -*-
import os
import pytest
from loguru import logger


if __name__ == '__main__':
    logger.info("Starting.........  走起 ......")
    # 命令行输入 'pytest'
    os.system('pytest')

    # 或者 输入 pytest.main(),效果一样
    # pytest.main()

```



就 **相当于执行如下命令**：

```
pytest -v -s  test_offer_doctor.py
```

### addopts参数说明

**-v: 输出调试信息：具体测试用例名称 **

**-s: 输出调试信息：包括print打印的信息**
-n=num：启用多线程或分布式运行测试用例。需要安装pytest-xdist插件模块

**-m**=标签：执行被@pytest.mark.标签名标记的用例

重点介绍一下 如何通过addopts参数 筛选用例执行？ 

```
# 举例
#pytest.ini 配置文件

[pytest]
;# 选中指定标记smoke执行
;addopts = -v -s -m smoke

;# 同时选中多个标记：只要标记'smoke or test'都执行
;addopts = -v -s -m 'smoke or test'


```



## pytest 中内置的标记

如果您想通过 筛选用例标签来执行用例，只需要 通过 `addopts`来设置要执行的标签，比如 `addopts = -v -s -m 'smoke or offer_serve'`，结果是只会执行带有 `smoke` 或者 `offer_serve` 标签的用例。

那么如何自定义标签呢？很简单，如下

```
# pytest.ini
markers =
    smoke:冒烟测试用例
    offer_serve: offer微服务下的接口
    single_api: 单接口参数校验用例
    p0: level p0 (优先级)
    p1: level p1 (优先级)
```



## 失败重跑

```
# test_001.py  用例文件

# 频率+次数：失败重跑，reruns=2 重跑次数；reruns_delay=2 频率
@pytest.mark.flaky(reruns=100, reruns_delay=10)  
# smoke标记
@pytest.mark.smoke
def test_01(login):
	pass

```

参考：https://www.osgeo.cn/pytest/reference.html#ini-options-ref



## 自定义用例执行顺序

可以通过第三方插件pytest-ordering实现自定义用例执行顺序

```
pip install pytest-ordering
```

### 执行优先级
0>较小的正数>较大的正数>无标记>较小的负数>较大的负数
第一个执行：@pytest.mark.run(order=1)
第二个执行：@pytest.mark.run(order=2)
第三个执行：无标记
第四个执行：@pytest.mark.run(order=-1)
第五个执行：@pytest.mark.run(order=-2)

![Snipaste_2023-03-03_15-09-14](http://biji.51automate.cn/blogs/img/Snipaste_2023-03-03_15-09-14.png)



## fixture

fixture正常的话是不需要()来调用的，它就等于return的值；这里return的是函数对象，所有它能使用()调用；fixture函数的参数只能是其他fixture，这里的work_id是pytest-xdist定义的一个fixture。





## pytest 中内置的标记



#### pytest.mark.parametrize：用例参数化的标记





#### pytest.mark.usefixtures：给测试类或模块设置测试夹具

```
# TestDome这个测试类的所有测试用例均执行my_fixture这个夹具
@pytest.mark.usefixtures('my_fixture这个夹具')
class TestDome:
    # 函数用例 指定测试夹具
    def test_02(self):
        print('----测试用例：test_01------')

    # 函数用例 指定测试夹具
    def test_03(self):
        print('----测试用例：test_02------')
```



usefixtures 标记一般用于给测试类下面的测试方法统一设置测试夹具

1. 在类声明上面加 @pytest.mark.usefixtures() ，代表这个类里面所有测试用例都会调用该 fixture
2. 可以叠加多个 @pytest.mark.usefixtures() ，先执行的放底层，后执行的放上层
3. 可以传多个 fixture 参数，先执行的放前面，后执行的放后面
4. 如果 fixture 有返回值，用 @pytest.mark.usefixtures() 是无法获取到返回值的，必须用传参的方式（方式一）
