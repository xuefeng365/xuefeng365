# 基于fastapi-vue2开发的前后端CURD项目模板

### 项目结构

![image-20231127221727991](http://biji.51automate.cn/blogs/img/202311272315461.png)

### 项目地址



## 启动项目

### 配置相关

#### 配置文件位置

后端在 `configs/ .env`
前端在 `.env.development /  .env.production / `

#### 后端

##### 配置后端

```
# .env文件
端口配置 8888

其中要使用【注册功能】，同时需要在后端 core/config.py Settings类中 配置前端的域名和端口
```

安装依赖

```
cd ./backend/app   # 进入到后端程序代码的根目录

# 也可以使用命令创建虚拟环境
python -m virtualenv venv     # 创建虚拟环境

# 安装依赖(确保在cd ./backend/app 目录下)
pip install -r requirements.txt --target=./venv/Lib/site-packages  -i  https://pypi.doubanio.com/simple
```

##### 启动后端

```
cd ./backend/app
pyhton main.py
```

顺手打开一下文档

```
# docs
http://localhost:8888/docs

# redoc
http://localhost:8888/redoc
```



#### 前端

node环境(安装和配置参考我其它文章)

![image-20231208111246038](http://biji.51automate.cn/blogs/img/202312081112859.png)

##### 配置前端

```
 .env.development 
 前端端口配置（和后端端口保持一致）
 VUE_APP_BASE_API='http://127.0.0.1:8888/api/v1/'
```

安装依赖

```
cd .\frontend\dashborad\
npm i  # 安装包
```

##### 启动前端

```
cd .\frontend\dashborad\
npm run dev
```

#### 账号:

| 角色     | 用户名 | 密码     |
| -------- | ------ | -------- |
| 管理员   | admin  | admin123 |
| 运维员   | opt    | opt123   |
| 普通用户 | user   | 123456   |



## 技术点

概述：先看一个简单的业务流动。

前端一个请求过来，前端如何找到页面视图 并 组装入参，将请求发出去?

![image-20231124110755364](http://biji.51automate.cn/blogs/img/image-20231124110755364.png)

![image-20231124110905873](http://biji.51automate.cn/blogs/img/image-20231124110905873.png)

![image-20231124111013968](http://biji.51automate.cn/blogs/img/image-20231124111013968.png)

![image-20231128105318571](http://biji.51automate.cn/blogs/img/202311281053244.png)



#### 基于 JWT+OAuth2实现登录功能

登录接口将 token 返回了, 同时存了一份在redis  , key以`user_login_token_`开头

前端登录后，存账号、密码、是否记住我、 生成的token 存到cookie（当然也可以存到本地缓存）

> 我这里是存到cookie,以处理cookie举例。建议封装好 处理token的方法
>
> ![image-20231205111417388](http://biji.51automate.cn/blogs/img/202312051114494.png)

![image-20231205114612971](http://biji.51automate.cn/blogs/img/202312051146125.png)

![image-20231205114420966](http://biji.51automate.cn/blogs/img/202312051144069.png)

请求拦截器每次请求前都读取一下cookie或本地缓存中的token数据，设置到请求头中，后端接口再获取请求头中的token去验证。

> 关于如何设置的请求头中，下面图片有展示 ，至于为什么要这么做。因为这样符合 JWT+OAuth2的规范， 会自动检验取出headers['Authorization'] 、 'Bearer' + ' ' 对应的token信息进行解码

```
config.headers['Authorization'] = 'Bearer' + ' ' + getToken()
```

![image-20231205104502199](http://biji.51automate.cn/blogs/img/202312051045770.png)

![image-20231205121438594](http://biji.51automate.cn/blogs/img/202312051214725.png)

![image-20231205121530326](http://biji.51automate.cn/blogs/img/202312051215494.png)



#### 权限验证

##### 登录验证：

token（JWT+OAuth2）

##### 菜单权限：

1. 登录后 `user/info` 获取包含角色的个人信息，没角色先提醒配置角色。

   ![image-20231215130553766](http://biji.51automate.cn/blogs/img/202312151305123.png)

2. 再`user/routers`连表查询出 用户 拥有的所有角色对应的全部 角色菜单。

   ![image-20231215122810714](http://biji.51automate.cn/blogs/img/202312151228232.png)



##### 按钮级别的权限控制（redis缓存-权限标识）

![image-20231218193723411](http://biji.51automate.cn/blogs/img/202312181937170.png)

以访问【用户列表】为例，

> 定义接口时 先设定好 权限标识，标识信息通常的做法是 模块+功能+请求方式
>
> ![image-20231218163658123](http://biji.51automate.cn/blogs/img/202312181636271.png)

前端访问到这个接口，具体是怎么鉴权的？

> 逻辑：找出指定标识（一个或多个）关联的所有角色，再查一下该用户拥有的所有角色，如果有共同角色，说明有权限，返回User对象。

![image-20231215212340953](http://biji.51automate.cn/blogs/img/202312152123727.png)

![image-20231215212629197](http://biji.51automate.cn/blogs/img/202312152126820.png)

```
# 以上关联查询sql
SELECT DISTINCT
	t_perm_label_role.role_id AS id
FROM
	t_roles,
	t_perm_label_role
	INNER JOIN t_perm_label ON t_perm_label.id = t_perm_label_role.label_id 
WHERE
	t_perm_label.label IN ('perm:user:get') 
	AND t_perm_label.STATUS IN (0) 
	AND t_roles.is_deleted = 0
	AND t_perm_label_role.is_deleted = 0;
```

前端怎么控制 列表或者按钮 是否显示呢？

具体可通过 控制标识（所属任一角色绑定了这个标识就有权限）或控制角色（有这个角色就有权限）

> 举例 用户列表：通过标识 控制列表展示
>
> ![image-20231218190105760](http://biji.51automate.cn/blogs/img/202312181901578.png)
>
> 举例：通过角色控制 按钮展示
>
> ![image-20231218193325631](http://biji.51automate.cn/blogs/img/202312181933826.png)

原版的若依框架，新增菜单的时候，可以选择新增对象，比如菜单或按钮，新增了按钮之后可以在对角色分配权限时，自由选择下不同按钮，列表权限。

![image-20231218203830054](http://biji.51automate.cn/blogs/img/202312182038287.png)

目前我这里将按钮 权限控制，操作起来灵活性相对麻烦一点点，但是功能可以实现，后期再优化。 更多功能可参考 >> https://blog.csdn.net/Keep__Me/article/details/132595597



#### 菜单功能 快速新增和删除（待更新）



#### 缓存功能实现

对不会频繁变动的数据（参数配置、全局字典、权限标识等等）进行缓存，达到减轻数据库压力的目的，同时提高效率。

导入pyhon中第三方 cachetools 库 ，通过@cached 装饰器 直接装饰到查询 参数配置和全局字典的方法上 即可实现缓存效果。

```
from cachetools import cached, TTLCache  # 可以缓存curd的函数，指定里面的key
```

![image-20231208123159774](http://biji.51automate.cn/blogs/img/202312081232217.png)

##### 技术实现之参数配置

观察下图代码，细心的你可以发现，这里我使用了双缓存（redis+cachetools），其实选择其中之一就可以，这里只是记录一下，方便灵活使用。

其中cachetools缓存时长做了配置（redis写死了15分钟），可以自行配置。

![image-20231213215427873](http://biji.51automate.cn/blogs/img/202312132154390.png)

![image-20231213213719656](http://biji.51automate.cn/blogs/img/202312132137508.png)

![image-20231213214148512](http://biji.51automate.cn/blogs/img/202312132141711.png)

##### 技术实现之全局字典

![image-20231213235642572](http://biji.51automate.cn/blogs/img/202312132356293.png)

![image-20231214000321012](http://biji.51automate.cn/blogs/img/202312140003476.png)

![image-20231213235703292](http://biji.51automate.cn/blogs/img/202312132357404.png)



#### 从登录接口说一下 封装和后端的路由（APIRouter）分发

前端 按照模块 统一请求接口 ，比如user模考，全部写到use.js 

![image-20231122171826156](http://biji.51automate.cn/blogs/img/image-20231122171826156.png)

后端 根据不同的模块，进行路由分发

![image-20231128103137661](http://biji.51automate.cn/blogs/img/202311281031441.png)

```
from fastapi import APIRouter, Depends, Header, Request, UploadFile, HTTPException
```

fastapi中的依赖注入[Depends](https://blog.csdn.net/NeverLate_gogogo/article/details/112472480)   https://blog.csdn.net/NeverLate_gogogo/article/details/112472480

常用的依赖注入方法 封装在 common/deps.py中

```
用户路由权限、get_current_user（从token获取用户数据，比如 用户id）、解析验证token有效性（check_jwt_token）、get_db （获取SQLAlchemy会话）、获取redis连接(redis = request.app.state.redis)、获取用户IP、邮件发送实例等等
```



#### 需要登录身份的接口 如何访问

这里以`user/info`为例，token 携带在headers中

![image-20231122180535630](http://biji.51automate.cn/blogs/img/image-20231122180535630.png)



![image-20231122192026160](http://biji.51automate.cn/blogs/img/image-20231122192026160.png)



![image-20231122193748809](http://biji.51automate.cn/blogs/img/image-20231122193748809.png)

生成token以及token的结构

![image-20231122192741264](http://biji.51automate.cn/blogs/img/image-20231122192741264.png)

 orm进行增删改查

![image-20231122195624437](http://biji.51automate.cn/blogs/img/image-20231122195624437.png)

贴一下 用户和角色的模型类

![image-20231122195838332](http://biji.51automate.cn/blogs/img/image-20231122195838332.png)

有一点需要注意：设计的所有的数据表里都有 以下三个列：` is_deleted、modifier_id、modifier_time`

![image-20231123101050626](http://biji.51automate.cn/blogs/img/image-20231123101050626.png)



因为很多模块都要对 通过orm对数据库进行增删改查，所以我们将 curd 操作封装到为一个公共基类，其它模块只需要继承基类，就可以直接调用公共方法；当然如果不满意，可以重写父类方法（包括 构造函数和其它方法）



python中继承了父类，如何直接调用父类方法？

```
class Parent:
	
    def greet(self):
        print("Hello from Parent class")

class Child(Parent):
    def greet(self):
        # 调用父类的方法
        super().greet()
        # 补充子类特有的功能
        print("Hello from Child class")

# 创建子类对象
child = Child()
# 调用子类的方法
child.greet()

```



python中继承了父类，如何重写父类方法？



同时使用orm操作数据库 之前 都要建立 该模块自己的模型类，这里以User功能下的`User模型类`为例，我们将 抽离出部分字段（其它模块也要用到）作为 公共模型基类（目前是放在common/base_class.py），供不同模块继承。

![image-20231123113151420](http://biji.51automate.cn/blogs/img/image-20231123113151420.png)

![image-20231123113209172](http://biji.51automate.cn/blogs/img/image-20231123113209172.png)



pydantic



git提交

```
# 教程： https://blog.csdn.net/weixin_45697805/article/details/106197919
git status  #查看工作区代码相对于暂存区的差别
git add .    #将当前目录下修改的所有代码从工作区添加到暂存区 . 代表当前目录 add后面一个空格然后输入点

# 将缓存区内容添加到本地仓库。--no-verify：不验证代码格式 ， -a -m 可以将暂存区的所有内容合并提交到本地仓库
git commit --no-verify -m "20231128备份"

# 拉下git最新代码，必须要执行，不然会覆盖掉别人刚提交过的代码。
git pull 
# 推送。  origin #是远程主机，master表示是远程服务器上的master分支，分支名是可以修改成其他分支 的名字的
git push origin master 


```

### 获取我的后端项目中查询出来的行数据中包含有datatime的字段，使用JSONResponse返回数据，会报错

> json序列化时会报错
>
> 了解更多 [FastAPI- JSONResponse](https://blog.csdn.net/qq_33801641/article/details/120600646)
>
> [关于fastapi不错的系列文章](https://www.w3cschool.cn/fastapi/fastapi-mfcs3ldi.html)
>
> 官方教程：https://fastapi.tiangolo.com/zh/learn/

目前是通过orm查询数据

有两种方案：

第一种，针对具体返回字段进行数据处理（比如 转为 时间字符串）再 通过 from fastapi.responses import JSONResponse 返回出去。

第二种，不使用JSONResponse 



#### 使用FastAPI和Celery的异步任务(更多内容可搜索：celery和redis fastapi)



#### FastApi结合loguru日志使用

https://blog.csdn.net/qq_51967017/article/details/134045236



关于跨域

![image-20231124150549572](http://biji.51automate.cn/blogs/img/image-20231124150549572.png)

#### 关于数据库

##### 连接池机制

> 避免频繁的创建、断开数据库连接造成的开销，提高性能。

session线程是不安全的，在多线程情况下，当多个线程共享一个session时，数据处理就会发生错误。为了保证线程安全，需使用scoped_session方法，它可以 返回一个可跨线程使用的局部会话。当多个线程使用会话时，会自动为每一个线程创建一个会话，用完会自动关闭。



事务管理

> 一系列操作的组合，要么全部正确的执行，要不全都不执行。进而保障数据的一致性。



使用 SQLAlchemy 的 ORM 模型时进行数据库操作，ORM 会在默认情况下自动开启事务，并且在会话结束时自动提交事务，如果出错了会自动回滚。

但是在执行原始的 SQL 语句时，需要手动管理事务的提交和回滚。像这样

```
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///:memory:')
Session = sessionmaker(bind=engine)
session = Session()

try:
    # 执行一些操作，可能会出现异常
    with session.begiin():
    	session.execute()
    	session.execute()
    # 提交事务
    session.commit()	
except:
    # 回滚
    session.rollback()

finally:
	session.colse()


```







原本

pymysql并不是线程安全的，所以当有多个线程同时使用pymysql操作数据库时，就会出现问题。

所以从线程安全和支持更高的并发的角度，我们一般会使用数据库连接池来解决，一般自己实现一个比较麻烦，通常是使用第三方库，如 `DBUtils`

发现sqlalchemy 除了orm操作数据库，还可以操作手写sql



尽管SQLAlchemy 有链接池， 既可以使用orm功能，也可以执行手写sql的功能， 但是我不想用它执行手写sql的功能，我可以使用其它工具代替吗？



[封装sqlalchemy手动执行sql增删改查自用](https://blog.csdn.net/Yellow_python/article/details/104172537)

[方法2 ：](https://www.jianshu.com/p/cf97d753b117)



java中使用 Druid 连接池操作数据库









---

我的项目

![image-20231128124655907](http://biji.51automate.cn/blogs/img/202311281247815.png)





---

#### 关于搜索 和时间相关

日期时间时区： https://juejin.cn/post/7277111344003186742

SQL SQLAlchemy orm：如何筛选日期字段(获取时间范围内的数据)

> 其中deadline_data中的列表数据 是时间字符串即可。可以是年月日 或者 年月日时分秒





拿**查询功能**来举例， 使用SQLAlchemy orm 如何进行时间大小比较或者一段时间范围内搜索。
比如数据表中的deadline（截止时间）类型是datetime, 

如果前端传时间戳给后端。
第一种情况：后端可以直接对比时间戳

> 比较大小

![image-20231130230150283](http://biji.51automate.cn/blogs/img/202311302301013.png)

> 时间范围内-- between（） 括号内的类型保持一致即可， 都是年月日 或者 年月日时分秒或者时间戳。

![image-20231129211001127](http://biji.51automate.cn/blogs/img/202311292110509.png)

不嫌麻烦你当然也可以将前端传入的时间戳转为时间字符串，同时将数据表里的datatime类型转为时间字符串再进行比较

```
from datetime import datetime
# 假设end_time_str是一个时间戳
end_time_timestamp = 1630863600  # 2021-09-05 12:00:00 的时间戳

# 将时间戳转换为datetime对象
end_time = datetime.fromtimestamp(end_time_timestamp)

# 将datetime对象格式化为时间字符串
end_time_str = end_time.strftime('%Y-%m-%d %H:%M:%S')
```

第二种情况： 前端传时间字符串，可以先转为时间戳再对比大小

```
from datetime import datetime

# 假设end_time是时间字符串
end_time_str = "2023-12-31 23:59:59"

# 将时间字符串转换为datetime类型
end_time = datetime.strptime(end_time_str, '%Y-%m-%d %H:%M:%S')

# 将datetime对象转换为时间戳
end_time_timestamp = end_time.timestamp()
```



现在控制输出模型并没有生效，后面要解决一下 。

### Post请求的3种编码格式：application/x-www-form-urlencoded和multipart/form-data和application/json



##### 1、application/x-www-form-urlencoded

1）浏览器的原生form表单
2） 提交的数据按照 key1=val1&key2=val2 的方式进行编码，key和val都进行了URL转码

如何发x-www-form-urlencoded请求

https://blog.csdn.net/u013258447/article/details/101107743





通过创建人id查出对应的账号名，显示再下拉选择中 



期间遇到了另外的问题 

比如 

1. 返回的字段的排序问题 如何能按定义模型类的字段顺序 输出。

   > ![image-20231129211244713](http://biji.51automate.cn/blogs/img/202311292112152.png)

2. deadline 是一个列表 ，如果我想 指定列表里的字典的key 比如 {'start_time' : '', 'end_time' : ''}

3. 定义好的 deadline 在文档里能清晰的展示出结构比如这样

   > ![image-20231129211719921](http://biji.51automate.cn/blogs/img/202311292117037.png)

   使用 FastAPI ，你可以定义、校验、记录文档并使用任意深度嵌套的模型（归功于Pydantic）

   https://fastapi.tiangolo.com/zh/tutorial/body-nested-models/#__tabbed_2_3





fastapi简单的demo 

https://www.cnblogs.com/Detector/p/17458890.html



---

#### 传递数据的方式

在 FastAPI 中，可以使用路径参数、查询参数、请求体参数和标头参数来传递数据。路径参数用于从 URL 中提取值，查询参数用于在 URL 中以查询字符串的形式传递值，请求体参数用于 POST、PUT 等请求的请求体中传递值，标头参数用于在 HTTP 标头中传递值。

对于 GET 请求，通常会使用查询参数来传递数据。查询参数可以通过查询字符串的方式直接附加在 URL 后面，因此非常适合用于 GET 请求。



请求体的作用 



请求体嵌套模型

```
from typing import List, Set, Union

from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()
    images: Union[List[Image], None] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
    
    
    
    
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": [
        "rock",
        "metal",
        "bar"
    ],
    "images": [
        {
            "url": "http://example.com/baz.jpg",
            "name": "The Foo live"
        },
        {
            "url": "http://example.com/dave.jpg",
            "name": "The Baz"
        }
    ]
}


```

常用的定义 可选参数的方式有 

```
from typing import Union, Optional

description: Union[str, int, None] = None  设置默认，就是可选，不设置就是必填。
description: Optional[str]  # 可选参数，Optional可以不设定默认值（默认为None），这里指定了 str类型,所以取值可以是str或者None


id: int = Form(...)  # 三个点 表示必填，这里指必填；并且没有设定默认值，当然你可以自己设置默认值
```







请求体参数：您可以使用 pydantic 的 Field 在pydantic内部（也就是我们定义的Schemas） 声明出入参类型和校验规则。

![image-20231130114814351](http://biji.51automate.cn/blogs/img/202311301148712.png)





在 路径函数中 使用**Query**更进一步 进行默认值、长度、正则、描述信息校验

> 通常情况下，FastAPI会自动解析`Query`、`Path`、`Body`等函数中的参数信息，并将其显示在生成的API文档中。

![image-20231130102644700](http://biji.51automate.cn/blogs/img/202311301026039.png)



```
def Query(  # noqa: N802
    default: Any = Undefined,
    *,
    alias: Optional[str] = None,
    title: Optional[str] = None,
    description: Optional[str] = None,  
    gt: Optional[float] = None,  # greater than  大于
    ge: Optional[float] = None,  # greater than or equal to  大于或等于
    lt: Optional[float] = None,  # less than 小于
    le: Optional[float] = None,  # less than or equal to   小于或等于
    min_length: Optional[int] = None,  # 最短
    max_length: Optional[int] = None,  # 最长
    regex: Optional[str] = None,
    example: Any = Undefined,
    examples: Optional[Dict[str, Any]] = None,
    deprecated: Optional[bool] = None,
    include_in_schema: bool = True,
    **extra: Any,

```

查询参数比较多的情况下，如果全部写在接口定义文件中，会变动很长。起初我是考虑将其放在 pydantic的参数模型中定义，然后接口文件中，导入模型使用即可，但是发现这样的操作的话，入参变成了请求体，并非预期的查询参数，很显然，正常情况下get请求我们使用查询参数更合适。

所以网上找到了一种方法：使用 Depends进行一次转化。

![image-20231204104925655](http://biji.51automate.cn/blogs/img/202312041049238.png)

![image-20231204105053369](http://biji.51automate.cn/blogs/img/202312041050771.png)

![image-20231204105245222](http://biji.51automate.cn/blogs/img/202312041052834.png)



发送表单数据

fastapi表单数据 

如果请求体中的数据类型是 json或者表单数据 ，应该怎么处理

https://deepinout.com/fastapi/fastapi-questions/71_fastapi_supporting_both_form_and_json_encoded_bodys_with_fastapi.html

常见的有原生表单（application/x-www-form-urlencoded）、form-data格式

目前表单格式定义，只能再接口定义文件中写，不能使用参数模型（使用了就会变成请求体参数-类型为json）

![image-20231204110833494](http://biji.51automate.cn/blogs/img/202312041108076.png)



返回与输入相同的数据

![image-20231130115032817](http://biji.51automate.cn/blogs/img/202311301150792.png)

输入使用一个模型，输出使用另一个模型。

![image-20231130115144640](http://biji.51automate.cn/blogs/img/202311301151847.png)

想要模型中有很多字段，这些字段不少是有默认值，响应返回的时候，其实我不想返回所有，因为这样会有很多没有实际数据 只有默认值的字段。我想忽略它们

```
response_model_exclude_unset=True
```

![image-20231130115616422](http://biji.51automate.cn/blogs/img/202311301156445.png)





fastapi（65）- 路由函数指定了 response_model，在返回自定义 JSONResponse 时， 不会限制它返回的数据结构...
https://blog.csdn.net/qq_33801641/article/details/121313492



枚举(找机会整理一下用法)

![image-20231203232646508](http://biji.51automate.cn/blogs/img/202312032326546.png)





查询参数、请求体参数 是否有必要使用 Pydantic模型指定参数类、 长度大小文档生成 等等 如果不使用会怎么样？

先看看请求体参数： https://juejin.cn/post/7068491099425210404

再看看查询参数





---

触发器 

```
// 计算 deadline 的值。触发器的逻辑可以使用 SQL 表达式来实现 first_active_time + reg_times 的计算，并将结果存储到 deadline 字段中

DELIMITER //

CREATE TRIGGER calculate_endtime_trigger
BEFORE INSERT ON t_reg_code
FOR EACH ROW
BEGIN
    SET NEW.deadline = DATE_ADD(NEW.first_active_time, INTERVAL NEW.reg_times DAY);
END;
//

DELIMITER ;



// 表 t_reg_code 中 字段first_active_time，有值了，就更新 字段 state的值为1

DELIMITER //

CREATE TRIGGER update_state_trigger
BEFORE UPDATE ON t_reg_code
FOR EACH ROW
BEGIN
    IF NEW.first_active_time IS NOT NULL THEN
        SET NEW.state = 1;
    END IF;
END;
//

DELIMITER ;

```



---





---

#### 若依框架---权限管理设计

http://wed.xjx100.cn/news/260010.html?action=onClick



---



![image-20231211001543332](http://biji.51automate.cn/blogs/img/202312110015707.png)

![image-20231211001138848](http://biji.51automate.cn/blogs/img/202312110011175.png)

另外 查询出来的字段太多了，而且还有空值的字段显示出来，如何自动去掉空值字段，并且可以方便的自定义排除的一些字段？

![image-20231211001127519](http://biji.51automate.cn/blogs/img/202312110011277.png)

----



orm 连表查 

定义不定义外键都可以连表查。

只不过orm模型定义了外键 可以更方便的反向查询；同时 使用join(不使用where) 会自动查找外键关联，不用像where一样必须写公共列

因为代码里只更多的是使用

使用外键（where和）和不使用外键(where 条件)

https://blog.csdn.net/weixin_40123451/article/details/117252473

https://www.cnblogs.com/shengs/p/10473524.html

https://www.cnblogs.com/drizzle-xu/p/10239437.html（不带外键关系）



---

#### 注册功能实现（邮箱验证）

防止重复注册、恶意注册（通过获取请求中的ip识别）、发送确认邮件等

![image-20231120195017783](http://biji.51automate.cn/blogs/img/image-20231120195017783.png)

![image-20231214101151328](http://biji.51automate.cn/blogs/img/202312141011684.png)

![image-20231214110309272](http://biji.51automate.cn/blogs/img/202312141103788.png)

目前存在一个小问题 需要更新

> 注册成功后要挂上默认角色；成功后，哈希密码 为空

#### fastapi结合alembic实现数据库热更+数据备份功能

##### 热更功能

可实现数据库结构管理（字段类型变更升级，加表，加项），可一键同步到数据库，我们只需要管理好 项目中的 models即可

> 实际的开发过程中，正常是通过维护数据模型 管理数据结构，比如添加了一个字段，维护字段类型等，然后 通过迁移 同步数据库。
>
> 同步过程中：不会重复建表。
>
> 模型结构变动就会更新； 注意只会更新结构，不会更新数据（需要自行备份数据-具体可参考：如何备份数据）。
>
> 迁移是单向的，只能模型到数据库；如果手动在数据库新增了 字段更改字段类型等，迁移操作并不会更新模型数据。

![image-20231212111622748](http://biji.51automate.cn/blogs/img/202312121116853.png)

![image-20231212111709848](http://biji.51automate.cn/blogs/img/202312121117805.png)



数据库热更之技术实现

> 结合 alembic ，执行 cmd命令。

```
# 先根据当前数据库状态（modle相关信息）自动【生成迁移脚本】 再 【执行迁移】
alembic revision --autogenerate -m "auto_update" && alembic upgrade head
python -m alembic revision --autogenerate -m "auto_update" && python -m alembic upgrade head

# 注意 多个命令是可以通过 && 连接，系统会按顺序一个一个执行。
```



执行 **alembic** 可能出现的问题 ：

1. 找不到alembic命令

   - 虚拟环境未激活

     ![image-20231212114246291](http://biji.51automate.cn/blogs/img/202312121142880.png)

     > pycharm如何激活虚拟环境 
     >
     > ![image-20231214125414766](http://biji.51automate.cn/blogs/img/202312141254336.png)

     

   - 环境激活了还是无法执行 alembic命令，试试命令前面加上 `python -m `

     ```
     python -m alembic revision --autogenerate -m "auto_update" && python -m alembic upgrade head
     ```

   

2. 成功执行**alembic**命令但是报错

   > 解决方案：删除迁移文件，重新生成，再次执行迁移即可（参考教程：https://www.jianshu.com/p/4aa58499e9d8）

   ![image-20231214124226128](http://biji.51automate.cn/blogs/img/202312141242801.png)

   同时 删除 数据库中 自动生成的表（里面存着version中迁移文件对应的版本号，不删的话不会再次生成，导致版本对应不上）

   ![image-20231212160320927](http://biji.51automate.cn/blogs/img/202312121603637.png)

   ![image-20231212161021499](http://biji.51automate.cn/blogs/img/202312121610544.png)

3. 如果项目没有 alembic文件夹 ，可自己初始化一个（目前用不到）

   ```
   参考步骤如下
   进入后端项目根目录（app目录下） 
   cd fastapi_vue\backend\app>
   
   初始化
   alembic init alembic
   
   找到生成的alembic.ini 文件，修改sqlalchemy.url
   sqlalchemy.url = mysql+pymysql://root:root@localhost:3306/fastapi_vue
   
   找到alembic文件夹下的env.py，修改target_metadata
   
   from apps.system.models.config_settings import ConfigSettings
   target_metadata = ConfigSettings.metadata
   
   生成迁移文件
   alembic revision --autogenerate -m "auto_update"
   
   执行迁移
   alembic upgrade head
   ```

----

##### 数据库数据备份

因为热更只能维护数据结构，表里的数据被没有同步，所以增加一个数据导出、导入功能（也可以使用**Navicat**导出导入sql文件）

可以使用**Navicat**导出导入sql文件（**推荐**）；也可以使用自己的脚本：

脚本技术实现：

> 核心原理将数据库中的表，导出成csv文件，需要的时候可以根据csv文件，插入到数据库中。

脚本操作过程中遇到的问题：

对于设置外键的关联表需要重点关注一下，插入数据时的执行顺序（基于这点，目前建议使用 Navicat，等我后期优化）

---

#### 定时任务功能

![image-20231213002954497](http://biji.51automate.cn/blogs/img/202312130029808.png)

![image-20231213003757216](http://biji.51automate.cn/blogs/img/202312130037013.png)

技术实现: 使用**apscheduler**调度任务

> 使用 `scheduler` **异步实例对象**来调度执行脚本时，实际上是将脚本封装成了一个异步函数，然后将其添加到调度器的任务队列中。这样，当调度器执行任务时，它会异步地调用这个异步函数，从而实现脚本的异步执行；同时我们可以对 `scheduler` 异步实例对象进行初始化操作，比如设置进程池的大小。

这里说一下项目的初始化包括的几个模块，跨域请求、日志、注册路由、redis、自定义异常（HTTPException）、**app事件监听**（startup、shutdown）、事件中做定时任务调度（开启和关闭）,我就是在app主程序的监听事件中，启动和初始化定时任务的。

下图是项目中生成 异步实例对象的方法

![image-20231214140328792](http://biji.51automate.cn/blogs/img/202312141403370.png)

尽管调度是异步的，但是如果要更好的提升执行效能，我们的定时任务脚本最好也是通过**异步编程**，这样才可以**异步执行**。具体可以使用`asyncio`库，进行异步编程。

```
# 异步编程

import asyncio

async def my_async_function():
    print("Hello, world!")
    await asyncio.sleep(1)
    print("Hello again!")
```



##### 遇到的问题

1. 每次重启项目后APScheduler中的任务就会从内存中删除，导致APScheduler中的任务和数据库任务状态不一致

   > 解决方案：项目启动时将数据库中任务状态初始化为**暂停状态**，再从数据库 查询出 所有开启的任务 重新调度执行 任务状态调整为 **运行中**。

   当然添加到调度管理器时，会处理一下逻辑，比如 任务不可重复添加，每个任务使用的执行策略（立刻执行、执行一次、放弃执行等等），是不是支持并发，cron表达式（指定定时任务执行时间），调度日志、钉钉或者邮件通知这些。

2. 不管任务是否被调度，前端页面，我想执行脚本，如果直接执行脚本，会卡住。

   > 解决方案：改为多线程执行。这样主线程可以继续执行其他任务，而脚本函数会在另一个线程中运行，不会影响主线程的执行。

   ![image-20231214143237513](http://biji.51automate.cn/blogs/img/202312141432258.png)

3. 脚本传参问题。

   scheduler.add_job() 本身支持 位置和字典传参
   所以我们的脚本方法写好后，具体参数，可以在前端页面直接配置。所以我们的脚本方法写好后，具体参数，可以在前端页面直接配置。

   ![image-20231214202447968](http://biji.51automate.cn/blogs/img/202312142024610.png)

   ```
   import time
   
   def print_numbers(start, end, *args, **kwargs):
       for i in range(start, end):
           print(i)
           time.sleep(1)
           
       for arg in args:
       	print(arg)
       
       for key, value in kwargs.items():
           print(f"{key}: {value}")
   
   print_numbers(1,10,name='zhangsan',age=20)
   ```

   

4. 000

5. 151



---

#### 文件导入导出功能

![image-20231212224622913](http://biji.51automate.cn/blogs/img/202312122246858.png)



基于importlib.import_module() 动态导入模块，实现导入导出文件功能。

##### 导出板块：

更进一步，可以根据 前端传过来的筛选条件，查询后台数据。通过`openpyxl`和二进制流工具类输出二进制信息，再通过fastapi响应（可指定headers），异步导出文件。更多自定义响应（Html、文件、流、其它）可参考官方文档： https://fastapi.tiangolo.com/zh/advanced/custom-response/?h=

技术实现（导入导出单独存放在一个**report**文件夹下，方便管理）：

![image-20231212201705027](http://biji.51automate.cn/blogs/img/202312122017390.png)

![image-20231212201907144](http://biji.51automate.cn/blogs/img/202312122019755.png)

![image-20231212202131058](http://biji.51automate.cn/blogs/img/202312122021596.png)

##### 导入板块（待完善）

----



## 项目部署

> 注意：
> 本源码中所有配置文件都使用 配置文件模板(.example)的形式上传, 目的是为了方便我自己的配置信息不被泄露。
> 部署项目时需要把.example后缀去掉才能使用。需要用到配置文件的地方均在后续说明有列出。

### FIRST

克隆项目主分支

```shell

```

数据库中创建DB

```sql
CREATE DATABASE fastapi_vue;  -- 仅供参考根据自己项目名和所用的数据库类型 修改SQL， 
```

运行脚本初始化数据库数据

```shell
python initial_data.py
```

或者直接导入初始化sql数据到数据库

```sql
USE fastapi_vue;
SOURCE fastapi_vue.sql;   -- 仅供参考
```

### APP

1. 安装python3、virtualenv、Nginx、 supervisor

```shell
# 略
```

2. 安装必要第三方库

```shell
cd ./fastAPI-vue/backend/app   # 进入到后端程序代码的根目录

python3 -m virtualenv venv     # 创建虚拟环境

source ./venv/bin/activate      # 进入虚拟环境

pip install -r requirements.txt   # 安装库  可使用谷内源：  -i https://pypi.tuna.tsinghua.edu.cn/simple
```

3. 准备程序配置文件 

```shell
cp ./configs/.env.example  ./configs/.env    # 复制配置模板

vim ./configs/.env     # 拷贝配置文件

# python main.py   # 测试项目是否成功运行，
```

> 根据需要修改.env的配置内容，配置所有的参数参考 ./core/config.py -> class Settings

4. 使用supervisor管理项目(生产环境)

```shell
cp ./configs/supervisor.conf.example  ./configs/supervisor.conf   # 拷贝配置文件

sudo ln -s /home/ubuntu/opt/fastAPI-vue/backend/app/configs/supervisor.conf /etc/supervisor/conf.d/fastapi_vue_backend.conf   # 配置文件软链到supervisor的配置文件目录， 此处目录路径仅供参考

vim ./configs/supervisor.conf    # 编辑配置文件,已有参考配置，按需修改

sudo supervisorctl update     # 更新supervisor

sudo supervisorctl start fastapi-backend:   # 启动项目
```

### WEB

1. 安装node、npm

```shell
# 略
```

2. 安装相关第三方包

```shell
cd ./fastAPI-vue/frontend/dashborad   # 进入项目目录

npm i  # 安装包
```

3. 开发环境配置

```shell
cp .env.development.example .env.development  # 复制配置文件

vim .env.development  # 编辑配置文件

npm run dev    # 测试运行程序 
```

4. 生产环境配置

```shell
cp .env.production.example .env.production  # 复制配置文件

vim .env.production  # 编辑配置文件

npm run build:stage  # 打包项目文件 (可以考虑在本地打包后把dies文件上传服务器部署)
```


### NGINX

使用Nginx反向代理后端项目和前端文件夹。

```shell
cd ./fastAPI-vue  # 进入项目根目录，此处目录仅供参加

cp ./nginx.conf.example  ./nginx.conf   # 拷贝配置文件

sudo ln -s /home/ubuntu/opt/fastAPI-vue/nginx.conf /etc/nginx/sites-enabled/fastapi_vue.conf   # 配置文件软链到Nginx的配置文件目录， 此处目录路径仅供参考

vim ./nginx.conf    # 编辑配置文件,已有参考配置，按需修改

nginx -t   # 检查Nginx配置文件 

nginx -s reload   # 重启Nginx
```





#### 部署



运行容器 redis 使用 https://blog.csdn.net/yu_hongrun/article/details/107577674
根据redis镜像 启动一个 redis容器，并设置映射端口。
安装redis 可视化工具 ：Redis Desktop Manager，测试连接



### 前端vue项目打包,稍后部署用

因为下面要进行nginx识别转发，我们在`env.production`中提前配置，所有前端项目的请求base_api。

![image-20231221145839597](http://biji.51automate.cn/blogs/img/202312211458854.png)

```
# 生产环境打包
npm run build:prod
```

![image-20231221145911246](http://biji.51automate.cn/blogs/img/202312211459230.png)

### 云服务器部署发布

使用`MobaXterm`连接云服务器，我这里使阿里云服务器：`centos`

[MobaXterm具体操作可查看往期的笔记内容。]()

**该步骤是最重要的一步，实现过程中一定要注意是在服务端操作还是在本地计算机上操作。若在服务器上操作，还要注意是使用root用户进行操作还是使用git用户进行操作。**

修改云服务器密码

```
passwd
# 然后输入两次密码
```

#### 卸载nginx

参考：https://www.python100.com/html/0S5UG8L956FT.html

```
查看版本
nginx -v
```



#### 安装nginx

因为我们用nginx作Web服务器，所以我们需要先安装nginx服务。具体步骤如下：

使用root用户远程登录阿里云服务器，使用yum命令进行安装。

```
# 安装nginx的依赖环境
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

# 下载nginx安装包。
wget -c https://nginx.org/download/nginx-1.10.1.tar.gz

# 将安装包解压到/usr/local目录下
tar -xvf nginx-1.10.1.tar.gz -C /usr/local

# 进入/usr/local目录，确认nginx解压到该目录下
cd /usr/local
ls

# 进入nginx-1.10.1目录，会发现该目录下有一个configure文件，执行该配置文件
cd nginx-1.10.1/
ls
./configure

# 编译并安装nginx。
make
make install

# 查找nginx安装目录
whereis nginx
# 或者
find / -name nginx.conf

# 进入nginx的【安装目录】
cd /usr/local/nginx
ls

# 由于nginx默认通过80端口访问，而Linux默认情况下不会开发该端口号，因此需要开放linux的80端口供外部访问
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT

# 进入安装目录（usr/local/nginx）的sbin 目录下，启动nginx。
cd sbin
./nginx

# 没有任何消息，代表启动成功。此时，便可以通过“公网IP+端口”的方式访问 http://xx.xx.xxx.xxx:80/ 进入nginx欢迎页面了
```

#### 配置nginx转发

如果想对这个项目用特定的子域名进行访问，就要先进行 域名解析，比如我这里是用阿里的

![image-20231221150721215](http://biji.51automate.cn/blogs/img/202312211507579.png)

![image-20231221150904872](http://biji.51automate.cn/blogs/img/202312211509862.png)

```
# 专门为项目创建一个【前端部署目录】
mkdir -p /home/www/fastapivue2

在nginx配置文件中，配置server服务（监听端口和项目部署的目录）
配置文件路径如下：/usr/local/nginx/conf/nginx.conf

修改完之后，重新加载配置文件，才生效
/usr/local/nginx/sbin/nginx -s reload
```

配置文件 `nginx.conf`, 配置文件内容如下：直接复制即可

```

#user  nobody;
worker_processes  1;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        # 如果没有备案域名，可以填写服务器公网ip，以后有了再改回来
        server_name  51automate.cn;

        location / {
            root    /home/www/hexo;
            index  index.html index.htm;
        }
    }

    server {
        listen       80;
        server_name  myfav.51automate.cn;

        location / {
            root    /home/www/myFavorite;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
        
        # 访问链接中出现 /api/ 就转发到本地的9090端口（该端口是 后端服务端口）
        location /api/ {
            proxy_pass   http://127.0.0.1:9090/;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
        server {
        listen       80;
        server_name  fv2.51automate.cn;

        location / {
            root    /home/www/fastapivue2;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
        
        # 访问链接中出现 /api/ 就转发到本地的8888端口（该端口是 后端服务端口）
        location /api/ {
            proxy_pass   http://127.0.0.1:8888/api/;
        }
    }
    
}

```



重点注意端口转发内容

```
location /api/ {
            proxy_pass   http://127.0.0.1:8888/;
        }
```

另外配置nginx之后要**重新加载**才生效。



拓展：nginx的其它常用命令

```
重启nginx
/usr/local/nginx/sbin/nginx -s reopen
停止服务
/usr/local/nginx/sbin/nginx -s stop

手动启动nginx服务
cd /usr/local/nginx       进入nginx安装目录
sudo /usr/local/nginx/sbin/nginx  管理员权限【启动nginx】
```



#### 前端文件部署

将打包好的前端文件 上传到项目部署目录中`/home/www/fastapivue2/`

```
/home/www/fastapivue2/
```

![image-20231221152339634](http://biji.51automate.cn/blogs/img/202312211523781.png)

#### 打包遇到的问题：

npm run build 之后生成的index.html页面打开为空白

https://blog.csdn.net/qq_44785351/article/details/129320814



### 后端项目部署

#### 部署目录

指定一个后端项目文件存放路径，我用的是root账号，这里我直接放在 `/root/fv2/app/`目录下 。

![image-20231221153259019](http://biji.51automate.cn/blogs/img/202312211533079.png)

#### 项目工程安装python虚拟环境

##### 安装  `virtualenv `工具包，并创建虚拟环境 并安装依赖

supervisor

##### 创建虚拟环境 并安装依赖

```
安装 virtualenv 
pip38 install virtualenv -i https://pypi.tuna.tsinghua.edu.cn/simple/

cd /root/fv2/app   # 进入到后端程序代码的根目录

python38 -m virtualenv venv     # 创建虚拟环境

source ./venv/bin/activate      # 进入虚拟环境

pip38 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/   # 安装库
```

安装  `supervisor `工具包，具体怎么做（待更新）



#### 启动后端项目

```
cd /root/fv2/app

# 激活虚拟环境
source ./venv/bin/activate

# 解决Linux关闭终端（关闭SSH等）后运行的程序或者服务自动停止【后台运行程序】
nohup python main.py

# 访问
http://fv2.51automate.cn/
```



后台运行 `nohup python main.py` 如果要修改后端内容，端口会一直被占用，无法更新最新内容

如何解决端口被占用

```
lsof -i:8888  或者 netstat -ap|grep 8888

kill -9  进程号(pid)
或者
kill -s  进程号(pid)
```



 遇到的问题：

##### nginx部署vue项目(除首页外全404)

history模式，要设置 不管访问哪个目录路径，都会访问根目录的index.html文件

```
location / {
            root   /home/ruoyi/projects/ruoyi-ui;
            try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }
```



https://blog.csdn.net/HD243608836/article/details/133706915

https://blog.csdn.net/sluck_0430/article/details/123659161





```
# 把先前的nginx容器删除
docker stop nginx
docker rm nginx

docker run -p 80:80 --name nginx \
    -v /data/docker/nginx/www:/usr/share/nginx/html \
    -v /data/docker/nginx/log:/var/log/nginx \
    -v /data/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /data/docker/nginx/conf/conf.d/:/etc/nginx/conf.d/ \
    -d nginx
    
    
# 重新加载 Nginx 的配置文件
docker exec -it nginx nginx -s reload
docker kill -s HUP nginx
```



#### Docker之nginx镜像

https://juejin.cn/post/6844904128150241287
