---
title: 爬虫系列：seleinum调用chrome长时间运行后出现崩溃
series: 爬虫
tags: [web自动化,web爬虫]
calegories: [爬虫]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---
## 爬虫系列：seleinum调用chrome长时间运行后出现崩溃(内存溢出)

![image-20230403112708761](http://biji.51automate.cn/blogs/img/image-20230403112708761.png)



## 场景

`selenium`和`chromedriver`做自动化爬虫的时候，如果你用到循环就会发现`chromedriver.exe`和浏览器进程越来越多，最后卡死，脚本停止运行。我每隔10分钟就要把脚本终止，然后再开始运行，只能达到“半自动”的效果。

查阅资料后得知：chrome浏览器有内存泄漏的情况也可能是浏览器缓存过大，越堆越多。selenium模拟浏览器会产生大量的临时文件，那如何解决这个问题呢？

我网上找了很多办法，有的用到JS，有的用kill去杀掉进程。最后发现最简洁/有效的办法

### 解决方案0：

调用refresh刷新一下即可，进程会自动消掉，内存也不会被吃完。脚本可以一直运行到天荒地老。

```
# 刷新页面，减少缓存
driver.refresh()
```



## 其他解决方案

### 解决方案1：

python清理浏览器缓存(chrome)

进入清理缓存页，通过元素定位清理缓存 https://www.freesion.com/article/3732592128/

方法1

```
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
driver = webdriver.Chrome()
driver.get('chrome://settings/clearBrowserData')
driver.find_element_by_xpath('//settings-ui').send_keys(Keys.ENTER)
```

方法2

```

from selenium import webdriver

driver = webdriver.Chrome()
# 设置隐式等待
driver.implicitly_wait(10)

# 清除缓存提示框
driver.get('chrome://settings/clearBrowserData')

# 2S 等待时间
time.sleep(2)

clearButton = driver.execute_script("return document.querySelector('settings-ui').shadowRoot.querySelector('settings-main').shadowRoot.querySelector('settings-basic-page').shadowRoot.querySelector('settings-section > settings-privacy-page').shadowRoot.querySelector('settings-clear-browsing-data-dialog').shadowRoot.querySelector('#clearBrowsingDataDialog').querySelector('#clearBrowsingDataConfirm')")

clearButton.click()

# driver.quit()

```

方法3

```
#coding=utf-8
from selenium import webdriver
from time import sleep
from selenium.webdriver.support.select import Select

class timing(object):
    def __init__(self):


        option = webdriver.ChromeOptions()
        option.add_argument("--user-data-dir=C:/Users/Administrator/AppData/Local/Google/Chrome/User Data")
        self.driver = webdriver.Chrome(chrome_options=option)
        self.driver.get('chrome://settings/clearBrowserData')

    def open(self):
        sleep(3)
        # 定位清除缓存按钮
        clearButton = self.driver.execute_script("return document.querySelector('settings-ui').shadowRoot.querySelector('settings-main').shadowRoot.querySelector('settings-basic-page').shadowRoot.querySelector('settings-section > settings-privacy-page').shadowRoot.querySelector('settings-clear-browsing-data-dialog').shadowRoot.querySelector('#clearBrowsingDataDialog').querySelector('#clearBrowsingDataConfirm')")
        # 赋值时间范围筛选的js
        js = 'return document.querySelector("body > settings-ui").shadowRoot.querySelector("#main").shadowRoot.querySelector("settings-basic-page").shadowRoot.querySelector("#basicPage > settings-section:nth-child(8) > settings-privacy-page").shadowRoot.querySelector("settings-clear-browsing-data-dialog").shadowRoot.querySelector("#clearFromBasic").shadowRoot.querySelector("#dropdownMenu")'
        # 查询时间范围
        Select(self.driver.execute_script(js)).select_by_value('4')
        sleep(3)
        # 清除缓存
        clearButton.click()
        sleep(6)
        # 关闭
        self.driver.close()


if __name__ == '__main__':
    test = timing()
    test.open()

```



### 解决方案2：

selenium调用Chrome本身就会出现内存泄漏，换用firefox浏览器https://blog.csdn.net/xpt211314/article/details/125050558



### 解决方案3：

尝试使用 无头模式 handless 解决，适合不需要页面的朋友（很显然不适合催单场景）



## 拓展： 

仅清除cookie信息：

```
driver.delete_all_cookies()
```

注意 selenium webdriver退出方式

```
`driver.close()`改为`driver.quit()`
```

> `close()方法`：可能是只关闭了当前网页，而未关闭chrome，导致大量chrome并发，占用内存直至卡死。
>
> `quit()方法`：关闭当前页面并退出浏览器和退出`webdriver`驱动。
