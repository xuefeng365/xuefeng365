---
title: aamt工具：介绍--专注于接口自动化
series: 接口自动化
tags: [aamt,测试]
calegories: [接口自动化]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---

# aamt

正式版源码：

## aamt简介

`aamt`是**A**pi **A**uto**M**ation **T**est的首字母缩写，是一款基于pytest测试框架的测试工具，集成了各种实用的第三方包和优秀的自动化测试设计思想，帮你快速实现自动化项目落地。

## 快速入门

### 安装

安装最新版本

```python
pip install aamt
```

指定版本安装

```python
pip install aamt==0.2.5
```

升级aamt

```python
pip install -U aamt
# 大写U
```

验证安装成功：

```python
aamt -V
# 大写V
```

### 新建项目脚手架

```
aamt startproject demo
```

带上`-venv`参数，可创建单个项目的Python虚拟环境，并在该项目的虚拟环境中安装tep：

```
aamt startproject demo -venv
```

外网速度慢，pandas可能安装失败，推荐用国内镜像

```python
pip --default-timeout=6000 install -i https://pypi.tuna.tsinghua.edu.cn/simple aamt
```



## 目录结构说明

api ：各模块接口文件

data：用例yaml数据

case：测试用例；

fixtures：Pytest fixture，自动导入；

common：个人封装 比如邮件、mysql、project项目各种路径、token信息读取等

report：测试报告；

conftest.py：Pytest挂载；

file：待上传的文件目录；

utils：工具包；

pytest.ini：Pytest配置；

main.py：执行入口；

requirements.txt：项目依赖；

项目结构说明.txt ；

### 用例集

在case目录下将测试用例按功能模块分成多个用例集：

```
case
  offer_doctor
	test_offer_doctor.py
  offer_nurse
	test_offer_nurse.py
  brand
	test_brand.py
```

### 测试用例

必须遵循用例解耦原则，每条用例都是单独可运行的。用例由3个文件组成，一个存放不同功能模块的api接口，一个存放test用例，一个文件存放纯粹的yaml数据 ：

```
api
  offer_doctor
  	offer_need_api.py
case
  offer_doctor
  	test_offer_doctor.py
data
  offer_doctor
    offer_doctor.yaml
```

1. 单独把api摘出来，是为了方便不能的功能模块使用这个api接口，直接导入使用即可
2. 用例只需要安装流程组装不同的业务逻辑
3. yaml文件里是参数化数据



**一条测试用例由鉴权信息、多个测试步骤和多个断言组成**：

```
@allure.title("采购商部分入库后（入库数量3）,供应商只能同步到的入库数据为3")
def test(xuefeng1_offer_doctor1_need, xuefeng2_offer_nurse1_receive):
    case_vars.put("token", login["token"])
    cache = TepCache(env_vars=env_vars, case_vars=case_vars)
    # 供货商领取需求订单
    xuefeng2_offer_nurse1_receive.receive_offer()
    # pytest原生断言
    assert 1==1 
    # 供货商发货
    xuefeng2_offer_nurse1_receive.nurse_ship()
    # pytest原生断言
    assert 1==1
    # 采购商入库
    xuefeng1_offer_doctor1_need.add_inbound()
    # pytest原生断言 
    assert 1==1 
    # 供货商同步入库数据
    xuefeng2_offer_nurse1_receive.nurse_sync_inboundQuantity()
    # pytest原生断言 下文有详细介绍
    assert 1==1 
```

`xuefeng1_offer_doctor1_need`、`xuefeng2_offer_nurse1_receive` 为 session 级别的 `fixture`

```
# conftest.py

@pytest.fixture(scope="session")
def xuefeng2_offer_nurse1_receive():
    return Nurse_receive("xuefeng2_nurse1_token")
```

具体的存放位置，前期可以先写在不同功能模块的test*.py文件里；后期根据情况，可以统一存放在case目录下，如下

![image-20230130170609361](http://biji.51automate.cn/blogs/img/image-20230130170609361.png)



>  用例运行时 项目里的 fixture 会自动导入，要**注意**的是  **fixture 不要重名**，否则可能导入的fixture并不是你想要的



### 测试步骤

测试步骤采用了 `@allure.step('xx')` , 比如发货功能中的**上传运单接口** 被我封装为一个函数  它的步骤描述就是**供货商上传运单**。

步骤的实现函数定义在api文件中的功能模块类 `Nurse_receive`中：

```
# nurse_need_api.py

class Nurse_receive(HttpClient):

	def __init__(self, token='xuefeng2_nurse1_token'):
		# 超类继承
		super().__init__()
        # 从配置文件里 读取最新的token文件
        self.token = Operate_token().read_token(section='Token', key=token)
        
	@allure.step('采购商填运单发货')
    def nurse_ship(amounts=[2, 3]):
    	method = 'post'
        url = '/offer/shipment/create'
        # url处理方法
        url = self.get_full_url(url, h=self.host)
        
        # 调用上传运单方法
        trackNo = self.upload_waybill(file_path)
        
        # 组装数据
        list_= []
        for amount in amounts:
            dic_ = {}
            dic_['trackNo'] = trackNo
            dic_['amount'] = str(amount)
            list_.append(dic_)
		
		# 通过缓存池保存数据，方便测试用例其他步骤断言
        self.cache.put('shipmentHistoryList', list_)
        
        body = {"takeOfferId":takeOfferId, "tracks":list_}
        ret = self.send(url=url, body=body, method=method, x_token=self.token)
        assert jmespath.search('code',ret) < 400, f'供货商发货 接口报错：{ret}'
        
     @allure.step('供货商上传运单')   
     def upload_waybill(file_path):
		method = 'post'
        url = '/offer/shipment/create'
        url = self.get_full_url(url, h=self.host)
        body = {"takeOfferId":takeOfferId, "trackNo":trackNos}
        ret = self.send(url=url, body=body, method=method, x_token=self.token)
        assert jmespath.search('code',ret) < 400, f'供货商上传运单 接口报错：{ret}'
        retrun jmespath.search('result',ret)
```

1. `Nurse_receive`类 中有多个 不同小功能的 实现方法

2. `Nurse_receive` 类 通过 `super().__init__()` 继承了 HttpClient的初始化方法，有能力的朋友可以自行重写

3. HttpClient的初始化方法里有 自动实例了一个 **缓存变量池**、自动读取了一个yaml配置文件里的**环境变量数据**（包含 host域名、账号密码、mysql数据库等信息）

4. HttpClient里还对 requests库 进行了二次封装，参照postman发送请求时的操作，通过入参`body_type` 即可控制 requests请求  json、文件以及文件流、表单数据，极大提升书写效率

5. HttpClient里还对 url 进行了二次封装，书写接口请求时 只需要填写`接口路径`,  再调用 `get_full_url` 即可实现多场景的`完整url`。

   

### 测试数据

测试数据有两种存在形式

1. 对于业务链路比较长的业务场景，做参数化会比较繁琐，工作过程中这类冒烟用例的测试数据 我是直接写在 `test_offer_doctor.py` 
2. 对于比较简单的接口，测试人员时间允许的情况下 可以讲测试数据写在yaml里，用于参数化，覆盖更多的比如边界值、异常等场景



### 测试标题

测试标题采用了`@allure.title("")`：

```
@allure.title('验证采购商发offer功能正常')
def test(xuefeng1_offer_doctor1_need):
```



## 变量

**环境变量**：在`resources/env_vars`下不同环境的yaml文件里预填变量，在`resources/aamt.ini`中激活某个环境，模块api.py的功能类`Nurse_receive`已经继承`HttpClient`类（会自动读取环境变量），可使用 `self.host`、`self.env_vars_data` 直接引用：

```
# nurse_need_api.py

class Nurse_receive(HttpClient):

	def __init__(self, token='xuefeng2_nurse1_token'):
		# 超类继承
		super().__init__()
		
	def aaa():
		# 直接引用 环境变量
		self.env_vars_data
		# 直接引用 域名
		self.host
		# 默认header头
		self.default_header
```

**用例变量**：在用例中引入`case_vars` fixture，不同步骤函数的数据通过`case_vars.put()`、`case_vars.get()`传递, 

当用例中下一个步骤的断言 需要用到 上一个步骤的数据（入参或者响应），可通过各个功能类中的**缓存变量池** `self.cache`来获取，存到`case_vars`中，随用随取。

```
@allure.title("从登录到下单支付")
def test(case_vars):
    case_vars.put("token", login["token"])
    cache = TepCache(env_vars=env_vars, case_vars=case_vars)

    Step("搜索商品", step_search_sku, cache)
    Step("添加购物车", step_add_cart, cache)


def step_search_sku(cache: TepCache):
    url = cache.env_vars["domain"] + "/searchSku"
    headers = {"token": cache.case_vars.get("token")}
    body = data("查询SKU")

    response = request("get", url=url, headers=headers, params=body)
    assert response.status_code < 400

    cache.case_vars.put("skuId", response.jsonpath("$.skuId"))
    cache.case_vars.put("skuPrice", response.jsonpath("$.price"))

def step_add_cart(cache: TepCache):
    url = cache.env_vars["domain"] + "/addCart"
    headers = {"token": cache.case_vars.get("token")}
    body = data("添加购物车")
    body["skuId"] = cache.case_vars.get("skuId")

    response = request("post", url=url, headers=headers, json=body)
    assert response.status_code < 400

```

## 接口关联

如上所述，通过想`HttpClient类`中的缓存变量池`cache`实现了不同的接口间的关联，上一个接口的响应，提取后存入`self.cache`，下一个接口的入参，从`self.cache`取值。

## 数据提取

`utils/http_client.py`封装了requests.Response，添加了jsonpath方法，支持简单取值：

使用 jmespath或者jsonpath 取值

```
import jmespath
# 响应：ret结果
code = jmespath.search('code',ret)
```

## 断言

采用Python原生的assert断言。16种常用断言如下：

```
import allure


@allure.title("等于")
def test_assert_equal():
    assert 1 == 1


@allure.title("不等于")
def test_assert_not_equal():
    assert 1 != 2


@allure.title("大于")
def test_assert_greater_than():
    assert 2 > 1


@allure.title("小于")
def test_assert_less_than():
    assert 1 < 2


@allure.title("大于等于")
def test_assert_less_or_equals():
    assert 2 >= 1
    assert 2 >= 2


@allure.title("小于等于")
def test_assert_greater_or_equals():
    assert 1 <= 2
    assert 1 <= 1


@allure.title("长度相等")
def test_assert_length_equal():
    assert len("abc") == len("123")


@allure.title("长度大于")
def test_assert_length_greater_than():
    assert len("hello") > len("123")


@allure.title("长度小于")
def test_assert_length_less_than():
    assert len("hi") < len("123")


@allure.title("长度大于等于")
def test_assert_length_greater_or_equals():
    assert len("hello") >= len("123")
    assert len("123") >= len("123")


@allure.title("长度小于等于")
def test_assert_length_less_or_equals():
    assert len("123") <= len("hello")
    assert len("123") <= len("123")


@allure.title("字符串相等")
def test_assert_string_equals():
    assert "dongfanger" == "dongfanger"


@allure.title("以...开头")
def test_assert_startswith():
    assert "dongfanger".startswith("don")


@allure.title("以...结尾")
def test_assert_startswith():
    assert "dongfanger".endswith("er")


@allure.title("正则匹配")
def test_assert_regex_match():
    import re
    assert re.findall(r"don.*er", "dongfanger")


@allure.title("包含")
def test_assert_contains():
    assert "fang" in "dongfanger"
    assert 2 in [2, 3]
    assert "x" in {"x": "y"}.keys()


@allure.title("类型匹配")
def test_assert_type_match():
    assert isinstance(1, int)
    assert isinstance(0.2, float)
    assert isinstance(True, bool)
    assert isinstance(3e+26j, complex)
    assert isinstance("hi", str)
    assert isinstance([1, 2], list)
    assert isinstance((1, 2), tuple)
    assert isinstance({"a", "b", "c"}, set)
    assert isinstance({"x": 1}, dict)
```

## 测试报告

allure下载地址： https://github.com/allure-framework/allure2/releases

解压后将bin目录添加到系统环境变量Path。

可以 执行以下命令

```
pytest --aamt-reports
```

或者 运行项目根路径`main.py`

![image-20230130194450349](http://biji.51automate.cn/blogs/img/image-20230130194450349.png)

就能一键生成Allure测试报告，并且会把请求入参和响应出参，记录在测试报告中。



## 身份鉴权

封装好了HttpClient，有时候调试或者写爬虫，并不是也不想写test用例的情况下 进行接口调用，我特意将登录token信息 保存在了ini文件里，接口调用的时候 只需在实例功能模块类的时候 传入 不同账号的token标识`Nurse_receive("xuefeng2_nurse1_token")`  

![](http://biji.51automate.cn/blogs/img/20230130191502.png)

起初的时候 会有顾虑，每次调用都要实例化吗？ 当然没必要如此，我们只需要将一个个功能模块的实例统一存放在一个conftest.py 里

![image-20230130192425665](http://biji.51automate.cn/blogs/img/image-20230130192425665.png)

借助pytest的fixture夹具，编写用例的时候就可以 重复 调用，即插即用。如下

![](http://biji.51automate.cn/blogs/img/20230130193145.png)



## 用例执行

### 串行

使用`pytest`命令即可执行用例。

如果想要生成测试报告，使用以下命令

```python
pytest --aamt-reports
```

### 并行

使用`pytest -n auto`，由pytest-xdist提供支持。

```
pytest -n auto --aamt-reports
```

## 特色功能

### fixtures自动导入

不是必须在conftest.py里面定义fixture。只要在fixtures目录下，创建以`fixture_`开头的文件，fixture都会自动加载到pytest中 。 

这样做的还有一个好处

> 不同的测试人员，维护自己的fixture_xx.py文件，在push代码时，可以高效的处理冲突，方便维护。

![image-20230131114647493](http://biji.51automate.cn/blogs/img/image-20230131114647493.png)



### 分布式环境下 全局只登录一次, 解决token复用问题

预置了`fixtures/fixture_xf.py`登录接口，且全局仅执行一次，解决token复用问题：

```
import pytest

from api.logintoken import Login_after

@pytest.fixture(scope="session", autouse=True)
def login_doctor1(aamt_context_manager, env_vars_data):
    	"""
        aamt_context_manager 可兼容pytest-xdist分布式执行的上下文管理器
        即便分布式执行用例，该login只会在整个运行期间执行一次
        """
    def produce_expensive_data(variable):
        # 这里的登录方法 ，抽离出来，1、方便换不同的账号登录 2、遇到前后台不同的方法登录，也可以有效的区分		开，统一管理。
        return Login_after(variable['xuefeng_doctor1'], 'xuefeng_doctor1_token').login_()
    
    return aamt_context_manager(produce_expensive_data, env_vars_data)
```

aamt的fixture.py 内置了 封装了`aamt_context_manager `fixture，可兼容pytest-xdist 分布式执行的上下文管理器，即便多个session 也只会执行一次登录，具体实现方法后面会详细介绍。

## 常用封装
`project.py`，项目基本信息，比如根目录路径。

```
import os
from loguru import logger


class Config:
    # 全局项目根目录
    # root_dir = os.path.dirname(os.path.dirname(os.getcwd()))
    root_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    # logger.info(f'全局项目根目录:{root_dir}')

    data_dir = os.path.join(root_dir, "data")
    file_dir = os.path.join(root_dir, "file")
```

`emailhelper.py` 封装了 offer365、qq、163等不同服务商发送邮件的方法

`mysqlhelper.py` 封装了 增删改查的方法，可快速上手。



## 工具包

fastapi_mock.py，示例应用。

```
import uvicorn
from fastapi import FastAPI, Request

app = FastAPI()


@app.post("/login")
async def login(req: Request):
    body = await req.json()
    if body["username"] == "dongfanger" and body["password"] == "123456":
        return {"token": "de2e3ffu29"}
    return ""


@app.get("/searchSku")
def search_sku(req: Request):
    if req.headers.get("token") == "de2e3ffu29" and req.query_params.get("skuName") == "电子书":
        return {"skuId": "222", "price": "2.3"}
    return ""


@app.post("/addCart")
async def add_cart(req: Request):
    body = await req.json()
    if req.headers.get("token") == "de2e3ffu29" and body["skuId"] == "222":
        return {"skuId": "222", "price": "2.3", "skuNum": "3", "totalPrice": "6.9"}
    return ""


@app.post("/order")
async def order(req: Request):
    body = await req.json()
    if req.headers.get("token") == "de2e3ffu29" and body["skuId"] == "222":
        return {"orderId": "333"}
    return ""


@app.post("/pay")
async def pay(req: Request):
    body = await req.json()
    if req.headers.get("token") == "de2e3ffu29" and body["orderId"] == "333":
        return {"success": "true"}
    return ""


if __name__ == '__main__':
    uvicorn.run("fastapi_mock:app", host="127.0.0.1", port=5000)
```

mitm.py，流量录制，做的不是很好，将就看看。

```
#!/usr/bin/python
# encoding=utf-8

# mitmproxy录制流量自动生成用例

import os
import time

from mitmproxy import ctx

project_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
tests_dir = os.path.join(project_dir, "tests")
# tests/mitm
mitm_dir = os.path.join(tests_dir, "mitm")
if not os.path.exists(mitm_dir):
    os.mkdir(mitm_dir)
# 当前时间作为文件名
filename = f'test_{time.strftime("%Y%m%d_%H%M%S", time.localtime())}.py'
case_file = os.path.join(mitm_dir, filename)
# 生成用例文件
template = """import allure
from utils.http_client import request


@allure.title("")
def test(env_vars):
"""
if not os.path.exists(case_file):
    with open(case_file, "w", encoding="utf8") as fw:
        fw.write(template)


class Record:
    def __init__(self, domains):
        self.domains = domains

    def response(self, flow):
        if self.match(flow.request.url):
            # method
            method = flow.request.method.lower()
            ctx.log.error(method)
            # url
            url = flow.request.url
            ctx.log.error(url)
            # headers
            headers = dict(flow.request.headers)
            ctx.log.error(headers)
            # body
            body = flow.request.text or {}
            ctx.log.error(body)
            with open(case_file, "a", encoding="utf8") as fa:
                fa.write(self.step(method, url, headers, body))

    def match(self, url):
        if not self.domains:
            ctx.log.error("必须配置过滤域名")
            exit(-1)
        for domain in self.domains:
            if domain in url:
                return True
        return False

    def step(self, method, url, headers, body):
        if method == "get":
            body_grammar = f"params={body}"
        else:
            body_grammar = f"json={body}"
        return f"""
    # 描述
    # 数据
    # 请求
    response = request(
        "{method}",
        url="{url}",
        headers={headers},
        {body_grammar}
    )
    # 提取
    # 断言
    assert response.status_code < 400
"""


# ==================================配置开始==================================
addons = [
    Record(
        # 过滤域名
        [
            "http://www.httpbin.org",
            "http://127.0.0.1:5000"
        ],
    )
]
# ==================================配置结束==================================

"""
==================================命令说明开始==================================
# 正向代理（需要手动打开代理）
mitmdump -s mitm.py
# 反向代理
mitmdump -s mitm.py --mode reverse:http://127.0.0.1:5000 --listen-host 127.0.0.1 --listen-port 8000
==================================命令说明结束==================================
"""
```



`client.py`，requests库二次封装，当然有基础的朋友也可以使用原生库

```
#!/usr/bin/python
# encoding=utf-8

import decimal
import json
import time

import allure
import jsonpath
import requests
import urllib3
from loguru import logger
from requests import Response

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)


def request(method, url, **kwargs):
    template = """\n
Request URL: {}
Request Method: {}
Request Headers: {}
Request Payload: {}
Status Code: {}
Response: {}
Elapsed: {}
"""
    start = time.process_time()
    response = requests.request(method, url, **kwargs)  # requests.request原生用法
    end = time.process_time()
    elapsed = str(decimal.Decimal("%.3f" % float(end - start))) + "s"
    headers = kwargs.get("headers", {})
    kwargs.pop("headers")
    payload = kwargs
    log = template.format(url, method, json.dumps(headers), json.dumps(payload), response.status_code, response.text,
                          elapsed)
    logger.info(log)
    allure.attach(log, f'request & response', allure.attachment_type.TEXT)
    return TepResponse(response)


class TepResponse(Response):
    """
    二次封装requests.Response，添加额外方法
    """

    def __init__(self, response):
        super().__init__()
        for k, v in response.__dict__.items():
            self.__dict__[k] = v

    def jsonpath(self, expr):
        """
        此处强制取第一个值，便于简单取值
        如果复杂取值，建议直接jsonpath原生用法
        """
        return jsonpath.jsonpath(self.json(), expr)[0]
```





关于aamt的更多技术细节，请在源码中一探究竟吧。也可以添加微信**xiaobangzhu007**，随时与我联系。

## 后续

**aamt小工具**后期将会根据实际业务场景进行微调，结构上不会做大的改动。


