# 爬虫系列：如何解决selenium打开网页加载时间过长的问题


> 遇到的情况如下：使用selenium打开网页后，即使页面已经加载出来可以操作了，但是浏览器地址栏旁边还是在转圈，后面的代码也一直无法往下执行，时间开销特别大。

**原因：**大部分原因是由于静态文件加载太慢或者外链的CDN挂了导致的



##  解决方案

### 1.设置WebDriver的页面加载超时时间 

通过`driver.set_page_load_timeout(5)`设置超时时间后页面不必全部加载完成就可以执行下一步

```
from selenium import webdriver
 
driver = webdriver.Chrome()
 
# 设置页面加载时间
driver.set_page_load_timeout(5)
 
start = time.time()
 
try:
    driver.get(driver.get('http://www.baidu.com'))
except:                                        # 捕获timeout异常
    driver.execute_script('window.stop()')     # 执行Javascript来停止页面加载 window.stop()
 
end = time.time()
# 页面加载所需时间
print(end-start)
 
el = driver.find_element("name", value='wd')
#对元素输入文本
el.send_keys('这是我的第一次尝试')
#找到搜索按钮并点击一次
driver.find_element("id", value='su').click()
```



### 2.修改WebDriver的页面加载策略

WebDriver支持的**三种**页面加载策略（normal、eager、none）

可通过 pageLoadStrategy的取值 来设置。

1. `normal` （**默认策略**）： 页面所有及所有静态资源都下载完（如css、图片、js等）都加载完。

2. `eager`： 等待初始HTML文档完全加载和解析，并放弃css、图像和子框架的加载，该策略无关样式表、图片和subframes的加载。

3. `none`: 仅等待初始页面下载完成即可。

   

具体【页面加载策略 - pageLoadStrategy】如何配置拓展里有展示。

另外提醒一点 不同的策略并不是万能的，爬取不同的网页可能需要不同的方案，大佬们请根据实际情况自行选择最优方案。



## 拓展

有时候运行脚本需要对 启用的浏览器对象进行自定义配置，比如`无头模式`、`页面加载策略`、`指定下载目录`、`设置浏览器缓存目录`、`设置User-Agent`、`启动前加载反爬JS，强力去除自动化特征`、`设置代理ip访问`，后续将会逐一介绍。



常见的配置参数我自己封装到了一个工具类里面（如下），导入即可使用。

```
from selenium import webdriver
# from fake_useragent import UserAgent


# ChromeOptions类是一个配置 chrome 启动是属性的类。通过这个类，可以为chrome配置参数
class Options:
    # ua = UserAgent()  # 创建User-Agent对象
    # useragent = ua.random

    def conf_options(self):

        # 配置ChromeOptions
        options = webdriver.ChromeOptions()

        # 页面加载策略 normal （默认）; eager; none
        options.set_capability('pageLoadStrategy', 'eager')
        # 不加载图片（如果设置【页面加载策略】异常，可以添加以下参数，提升运行速度）
        options.add_argument('blink-settings=imagesEnabled=false')

        # 默认启动的driver窗体最大化
        options.add_argument('start-maximized')
        # 以最高权限运行 (解决DevToolsActivePort文件不存在的报错)
        options.add_argument('--no-sandbox')
        # 关闭“DevTools listening on ws://127.0.0.1:12345...........”的日志输出
        options.add_experimental_option('excludeSwitches', ['enable-logging'])
        # ERROR:ssl_client_socket 不安全的地址错误，循环报错，导致程序终止。以下参数可忽略掉那些证书错误
        options.add_argument('--ignore-certificate-errors')
        # 彻底禁用gpu加速启动
        options.add_argument('--disable-gpu --disable-software-rasterizer')
        # # 无痕模式
        # options.add_argument('incognito')

        # # 无头模式。
        # options.add_argument('--headless')

        # ----------------- 反爬措施 ----------
        # 去掉提示正在执行自动化的警告条
        options.add_experimental_option('useAutomationExtension', False)
        options.add_experimental_option('excludeSwitches', ['enable-automation'])

        # # 设置User-Agent
        # options.add_argument(f'User-Agent={self.useragent}')

        # # 添加本地缓存目录（指定）
        # 输入 chrome://Version  查看浏览器缓存文件
        # options.add_argument(r'--user-data-dir=C:\Users\Administrator\AppData\Local\Google\Chrome\User Data\Default')

        # 代理ip
        # options.add_argument('--proxy-server=http://ip:port')
        
        

        # ----------------- 反爬措施 ----------

        prefs = {}
        # 添加去掉密码弹窗管理（chrome密码登录时弹出的密码提示框，遮挡了要点击的元素，导致元素不可点击）
        prefs.update({"credentials_enable_service": False ,"profile.password_manager_enabled": False})

        # # -------------- 下载相关 -----------
        # # 设置下载目录(当前目录，可设置其他目录)
        # download_path = './'
        # prefs.update({"download.default_directory": download_path})
        # # 开启自动下载
        # prefs.update({"download.prompt_for_download": False})
        # # 取消浏览器下载时保存路径弹框
        # prefs.update({"download.directory_upgrade": True})
        # # -------------- 下载相关 -----------

        # # 是禁止弹出所有窗口(慎用)
        # prefs.update({"profile.default content settings·popups": 0})

        # 去掉"此文件类型可能会损害您的计算机”的提示
        prefs.update({"safebrowsing.enabled": True})
        options.add_experimental_option("prefs", prefs)

        # # 设置语言
        # options.add_argument('lang=en_US')

        return options



```

如何加载上面封装的配置项到webdriver对象？

```
# conftest.py

@pytest.fixture(scope="session")
def driver1():
    chrome_options = Options().conf_options()
    driver = webdriver.Chrome(options=chrome_options, executable_path=r'.\chrome\chromedriver.exe')
    yield driver
    print("全部用例执行完后 teardown quit dirver")
    driver.quit()
```



