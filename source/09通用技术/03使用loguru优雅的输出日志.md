---
title: 使用loguru优雅的输出日志
---
# 使用loguru优雅的输出日志

### 快速上手

##### 安装

```python
pip install loguru
```

##### 初次使用

```python
from loguru import logger

logger.debug("That's it, beautiful and simple logging!")
```

### 进阶用法

##### 配置日志格式

```python
import sys
from loguru import logger

#配置日志格式、默认日志级别level
logger.add(sys.stderr, format="{time} {level} {message}", filter="my_module", level="INFO")

logger.info("That's it, beautiful and simple logging!")
logger.debug("That's debug")
logger.warning("That's warning")
```

##### 日志文件

如果要将记录的消息保存到文件，则只需使用字符串路径作为接收器。

```python
logger.add("file_{time}.log")
```

##### 日志文件分隔、保留时间、压缩格式

如果需要设置日志文件大小、删除较旧的日志文件或希望节省存储空间，并设置压缩格式，这些都可以进行配置，而且配置起来非常简单。如下

```python
# rotation参数
logger.add("file_{time}.log", rotation="500 MB")  # 超过500MB 新创建一个log 文件
logger.add("file_{time}.log", rotation="12:00")   # 每天中午12点创建一个新的日志文件
logger.add("file_{time}.log", rotation="1 week")  # 每隔一个周新创建一个log 文件

# retention参数
logger.add("file_X.log", retention="10 days")  # 日志文件最长保留 10 天

# compression参数
logger.add("file_Y.log", compression="zip")  # 使用zip文件格式保存
```

> 保留时长也可以是一个 datetime.timedelta 对象，比如设置日志文件最多保留 5 个小时

```python
import datetime
from loguru import logger

logger.add('runtime_{time}.log', retention=datetime.timedelta(hours=5))
```

##### 异常追溯(Traceback捕获)

您是否曾经看到程序意外崩溃而没有在日志文件中看到任何内容？您是否注意到没有记录线程中发生的异常？这可以使用[`catch()`装饰器/上下文管理器解决，该管理器可以确保将任何错误信息正确保存到`logger`中。配置如下：

```python
import sys
from loguru import logger

@logger.catch
def my_function(x, y, z):
    # An error? It's caught anyway!
    return 1 / (x + y + z)
 
my_function(1,-1,0)
```

##### 色彩斑斓的日志(目前用不到)

如果您的终端兼容，Loguru会自动为日志添加颜色。您可以通过使用接收器格式来自定义自己喜欢的样式。配置方式如下：

```python
logger.add(sys.stdout,
    colorize=True,
    format="<green>{time}</green> <level>{message}</level>")

logger.add('logs/z_{time}.log',
           level='DEBUG',
           format='{time:YYYY-MM-DD :mm:ss} - {level} - {file} - {line} - {message}',
           rotation="10 MB")
```

##### 异步写入日志

- 默认情况下，添加到 `logger` 中的日志信息都是线程安全的。但这并不是多进程安全的，我们可以通过添加 `enqueue` 参数来确保日志完整性。
- 如果我们想要在异步任务中使用日志记录的话，也是可以使用同样的参数来保证的。并且通过 `complete()` 来等待执行完成。

```python
# 异步写入
logger.add("somefile.log", enqueue=True)
```

##### 配置编码格式-encoding

```csharp
logger.add(log_file_path, rotation="500 MB", encoding='utf8')
```

#####  通知机制

```python
import notifiers

params = {
    "username": "you@gmail.com",
    "password": "abc123",
    "to": "dest@gmail.com"
}

# Send a single notification
notifier = notifiers.get_notifier("gmail")
notifier.notify(message="The application is running!", **params)

# Be alerted on each error message
from notifiers.logging import NotificationHandler

handler = NotificationHandler("gmail", defaults=params)
logger.add(handler, level="ERROR")
```

### 要点解析

介绍，主要函数的使用方法和细节 - add()的创建和删除

**add() - 非常重要的参数 `sink` 参数**

* 可以传入一个 `file` 对象

  * 例如 `sys.stderr` 或者 `open('file.log', 'w')` 都可以

* 可以直接传入一个 `str` 字符串或者 `pathlib.Path` 对象

  * 代表文件路径，会自动创建对应路径的日志文件并将日志输出进去

* 另外添加 `sink` 之后我们也可以对其进行删除，相当于重新刷新并写入新的内容。删除的时候根据刚刚 `add` 方法返回的 `id` 进行删除即可。

  ```python
  from loguru import logger
  
  trace = logger.add('runtime.log')
  logger.debug('this is a debug message')
  logger.remove(trace)
  logger.debug('this is another debug message')
  ```

  可以发现，在调用 `remove` 方法之后，确实将历史 `log` 删除了。但实际上这并不是删除，只不过是将 `sink` 对象移除之后，在这之前的内容不会再输出到日志中，这样我们就可以实现日志的刷新重新写入操作

### 参考

官方文档：   https://loguru.readthedocs.io/en/stable/overview.html

官方github:  https://github.com/Delgan/loguru