# Fiddler

## 原理

- 代理就是在客户端和服务器之间设置一道关卡，客户端先将请求数据发送出去后，代理服务器会将数据包进行拦截，代理服务器再冒充客户端发送数据到服务器；同理，服务器将响应数据返回，代理服务器也会将数据拦截，再返回给客户端。
- Fiddler可以抓取支持http代理的任意程序的数据包，如果要抓取https会话，要先安装证书。
- 由于fiddler并不能对ios进行抓包，这里选用Charls总结可搜索Charls查看

## 安装-汉化版

阿里云盘：效率工具>办公

## 配置

### 1、设置链接和监听端口

![image-20230725101055341](http://biji.51automate.cn/blogs/imgimage-20230725101055341.png)


### 2、设置抓取https请求

默认fiddler 不抓取https的包，先设置https，就能抓取了

![image-20230925101629630](http://biji.51automate.cn/blogs/imgimage-20230925101629630.png)

![image-20230925101707678](http://biji.51automate.cn/blogs/imgimage-20230925101707678.png)

### 3、安装https证书（重点）

- PC端安装根证书![image-20230725101416014](http://biji.51automate.cn/blogs/imgimage-20230725101416014.png)![image-20230725101447430](http://biji.51automate.cn/blogs/imgimage-20230725101447430.png)![image-20230725101508377](http://biji.51automate.cn/blogs/imgimage-20230725101508377.png)![image-20230725101540214](http://biji.51automate.cn/blogs/imgimage-20230725101540214.png)

  

  注意：如果非首次安装或者抓包过程中出现各种抓不到等疑难杂症，建**议删除所有之前安装过的根证书** ，重新安装一次；重启fiddler；关闭电脑的防火墙，比如我把360的安全卫士关闭

  以下方法可以打开证书管理器可查看所有证书

  ![image-20230918143316025](http://biji.51automate.cn/blogs/imgimage-20230918143329397.png)![image-20230918143344343](http://biji.51automate.cn/blogs/imgimage-20230918143344343.png)



- 客户端-手机端 设置代理并安装证书

  查看本机ip

  方法1-通过fiddler直接查看![image-20230725103207730](http://biji.51automate.cn/blogs/imgimage-20230725103207730.png)

  方法2-通过CMD-输入`ipconfig`命令查看

  ![image-20230725103313763](http://biji.51automate.cn/blogs/imgimage-20230725103313763.png)

  不同客户端安装证书举例

  1. 真机-安卓

     华为手机为例

     > 第一次安装 需要设置锁屏密码，按提示设置即可

     - 要使用**手机自带的浏览器**下载证书

       ![image-20230925104334945](http://biji.51automate.cn/blogs/imgimage-20230925104334945.png)

     - 设置-安全-更多安装设置-加密和凭据-从存储设备安装-CA证书![image-20230925104614043](http://biji.51automate.cn/blogs/imgimage-20230925104614043.png)![image-20230925104820312](http://biji.51automate.cn/blogs/imgimage-20230925104820312.png)
     - 查看下载任务详情-通过文件管理器打开证书-进而确定 文件的下载路径。（有些手机会自动搜索出刚刚下载的证书，有些需要自己找一下）![image-20230925105745851](http://biji.51automate.cn/blogs/imgimage-20230925105745851.png)

  2. 模拟器

     > 第一次安装 需要设置锁屏密码，按提示设置即可

     输入 192.168.0.172:8888 下载证书，然后安装，安装步骤如下

     ![image-20230925110210854](http://biji.51automate.cn/blogs/imgimage-20230925110210854.png)![image-20230925110323445](http://biji.51automate.cn/blogs/imgimage-20230925110323445.png)

     

  3. 浏览器

     输入 192.168.0.172:8888 下载证书，然后安装，安装步骤如下

     ![image-20230925110834490](http://biji.51automate.cn/blogs/imgimage-20230925110834490.png)

     如果遇到PC浏览器抓不了包，同样是在这里将其删除![image-20230925114956171](http://biji.51automate.cn/blogs/imgimage-20230925114956171.png)

  4. 真机-IOS

     教程：https://blog.csdn.net/weixin_40608713/article/details/114873070

---

小米手机安装证书: https://blog.csdn.net/zpl123456/article/details/104071611
oppo手机安装证书: https://blog.csdn.net/plychoz/article/details/80832405

苹果手机安装证书: https://blog.csdn.net/dlrb_beautiful/article/details/120067364

---

## 实用技巧（重要）

### 过滤接口信息

1. 从非浏览器抓包-适用app/小程序![image-20230725100823521](http://biji.51automate.cn/blogs/imgimage-20230725100823521.png)

2. 按主机过滤![image-20230725102412814](http://biji.51automate.cn/blogs/imgimage-20230725102412814.png)

   注意 一般app 可能会调用多个域名或者多个子域名, 这里就要用到通配符匹配，比如

   ```
   # 通配符，不同域名之间用英文 ; 分开
   *.aaa.com;*.bbb.com; 
   ```

3. 类型过滤

   ```
   # 通过 正则匹配 将各种图片、CSS、JS这类的静态素材,直接过滤掉。
   REGEX:(?i)/[^\?/]*\.(css|ico|jpg|png|gif|bmp|wav|js|jp?g|swf|woff)(\?.*)?$;
   ```

4. 按进程抓包--只抓取该指定进程下的接口信息![image-20230725110624183](http://biji.51automate.cn/blogs/imgimage-20230725110624183.png)

5. 状态码过滤

   ![image-20230725112051946](http://biji.51automate.cn/blogs/imgimage-20230725112051946.png)![image-20230925121513173](http://biji.51automate.cn/blogs/imgimage-20230925121513173.png)

6. 过滤connect连接（Https接口）

   ![image-20230725121214512](http://biji.51automate.cn/blogs/imgimage-20230725121214512.png)

   > 在抓取https的数据包时，会显示很多`Tunnel to….443`的接口信息，其实不必担心，https链接前，先会使用`connect`方法进行握手，建立好连接后，再进行通信，我们可以选择将这类信息进行隐藏。

7. 按URL进行过滤

   场景1：调试网页的时候,可能某些频繁的ajax轮询请求会干扰我们，可以屏蔽某些url的抓取。

   场景2：目前我只是抓完包之后，观察结果时，用这个方法去掉部分url干扰。

   > 但是有个缺点，只对当前列表已有的这个url进行屏蔽，后面新抓到的即便是相同的url也不再生效；而且重新打开fiddler屏蔽词也会失效。

   ![image-20230925120319065](http://biji.51automate.cn/blogs/imgimage-20230925120319065.png)

   ![image-20230925120515301](http://biji.51automate.cn/blogs/imgimage-20230925120515301.png)

   

### 弱网设置

限速规则设置(**ctrl+F 搜 300**)

![image-20230725111348600](http://biji.51automate.cn/blogs/imgimage-20230725111348600.png)



根据需求自行修改：

> [参考教程1](https://blog.csdn.net/qq_27658681/article/details/112918188)

上传1KB需要300ms，转化一下：1Kb/0.3s = 10/3=3.3(KB/s)

备注：每次编辑并保存配置文件后，Simulate Modem Speeds选项会被取消，需要**重新勾选**。

![image-20230725112504012](http://biji.51automate.cn/blogs/imgimage-20230725112504012.png)



### 拦截修改数据（待整理）

#### 断点

断点设置

https://www.yuque.com/liruijie-j3set/bbczmp/ko19o3

https://www.yuque.com/duoli/develop/software-fiddler-usage



#### Composer组合器

> 模拟向服务器发送数据的过程（拦截修改数据)

![img](http://biji.51automate.cn/blogs/img20200726145155260.png)![img](http://biji.51automate.cn/blogs/img20200726145212335.png)

### 统计（待整理）



## 其它的疑难杂症 

### 无网络或者有网络也抓不到包

当然这里讲的 并非没有正确的安装https证书。

#### app不走代理

> APP使用了默认禁用系统代理

正常情况下 各类应用会先检查是不是有系统代理，如果有就直接用。但是部分应用（比如我遇到的微信小程序-游戏）在遇到系统设置的代理的时候，会绕过代理去发送请求 或者 直接报错，这样的后果就是要么App无法连接网络，没有经过Fiddler等软件的代理，自然也无法正常抓包。

#### app代理检测

游戏app, 通过代理网络访问：初始化失败，经过观察也就初始化接口是有校验是不是通过代理进入游戏。

> 解决方案：可以尝试先连接其它非代理网络，成功初始化之后，再连接代理网络，尽可以成功抓包了

#### 微信小程序抓不包

> 具体表现为
>
> 1、根本进不去游戏，2、即便非代理网络先进去了，再切换回代理网络，正常使用小程序，还是抓不到包。

解决方案：可以使用Windows PC 版 进行小程序抓包（亲测可用）。如果使用 PC最新版本的微信客户端还是抓不到 ，可以使用旧版。

阿里云盘：搜索 微信旧版本

#### 加密数据包破解（逆向）- 脱壳、找到解密函数、重现

这部分内容是高阶操作，[具体可参考1](https://blog.csdn.net/shayuchaor/article/details/83746457#:~:text=Android%E9%80%86%E5%90%91%E5%AE%9E%E6%88%98%E7%AF%87%EF%BC%88%E5%8A%A0%E5%AF%86%E6%95%B0%E6%8D%AE%E5%8C%85%E7%A0%B4%E8%A7%A3%EF%BC%89%201%201.%20%E5%AE%9E%E6%88%98%E8%83%8C%E6%99%AF%20%E7%94%B1%E4%BA%8E%E5%B7%A5%E4%BD%9C%E9%9C%80%E8%A6%81%EF%BC%8C%E8%A6%81%E7%88%AC%E5%8F%96%E6%9F%90%E6%AC%BEApp%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8CApp%E7%9A%84%E5%85%B7%E4%BD%93%E5%90%8D%E7%A7%B0%E6%AD%A4%E5%A4%84%E4%B8%8D%E4%BE%BF%E9%80%8F%E9%9C%B2%EF%BC%8C%E9%81%BF%E5%85%8D%E4%BB%96%E4%BB%AC%E5%8F%91%E7%8E%B0%E5%B9%B6%E4%BF%AE%E6%94%B9%E5%8A%A0%E5%AF%86%E9%80%BB%E8%BE%91%E6%88%91%E5%B0%B1%E5%BE%97%E9%87%8D%E6%96%B0%E7%A0%B4%E8%A7%A3%E4%BA%86%E3%80%82%20%E7%88%AC%E5%8F%96%E8%BF%99%E6%AC%BEApp%E6%97%B6%E5%8F%91%E7%8E%B0%EF%BC%8C%E6%8A%93%E5%8C%85%E6%8A%93%E5%88%B0%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%AF%E5%8A%A0%E5%AF%86%E8%BF%87%E7%9A%84%EF%BC%8C%E5%A6%82%E5%9B%BE1%E6%89%80%E7%A4%BA%EF%BC%88%E5%8E%9F%E6%95%B0%E6%8D%AE%E8%BE%83%E9%95%BF%EF%BC%8C%E5%9B%BE%E4%B8%AD%E6%9C%89%E7%9C%81%E7%95%A5%EF%BC%89%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E8%BF%99%E4%B8%AA%E8%B6%85%E9%95%BF%E7%9A%84data1%E5%AD%97%E6%AE%B5%EF%BC%8C%E8%80%8C%E4%B8%94%E6%98%AF%E5%8A%A0%E5%AF%86%E8%BF%87%E7%9A%84%E3%80%82%20...%202,%E6%8B%BF%E5%88%B0APK%E5%90%8E%EF%BC%8C%E8%A6%81%E5%81%9A%E7%9A%84%E7%AC%AC%E4%B8%80%E4%BB%B6%E6%98%AF%E5%B0%B1%E6%98%AF%E6%9F%A5%E5%A3%B3%E3%80%82%20...%204%204.%20%E6%80%BB%E7%BB%93%20%E4%B8%8A%E8%BF%B0%E8%BF%87%E7%A8%8B%E5%B9%B6%E4%B8%8D%E5%A4%8D%E6%9D%82%EF%BC%8C%E4%B9%9F%E5%87%A0%E4%B9%8E%E6%B2%A1%E6%9C%89%E4%BB%BB%E4%BD%95%E9%9A%BE%E7%82%B9%EF%BC%8C%E5%9B%A0%E6%AD%A4%E8%BF%99%E7%A7%8D%E5%8A%A0%E5%AF%86%E6%96%B9%E5%BC%8F%E5%8F%AF%E4%BB%A5%E8%AF%B4%E5%B9%B6%E4%B8%8D%E5%90%88%E6%A0%BC%EF%BC%8C%E7%94%9A%E8%87%B3%E5%8F%AF%E4%BB%A5%E8%AF%B4%E6%9C%89%E4%BA%9B%E8%87%AA%E6%AC%BA%E6%AC%BA%E4%BA%BA%E3%80%82%20%E5%8A%A0%E5%AF%86%E7%9A%84%E6%9C%AC%E6%84%8F%E6%98%BE%E7%84%B6%E6%98%AF%E4%B8%BA%E4%BA%86%E5%A2%9E%E5%8A%A0%E4%B8%80%E7%82%B9%E7%A0%B4%E8%A7%A3%E9%9A%BE%E5%BA%A6%EF%BC%8C%E4%BD%86%E6%88%91%E8%BF%99%E6%A0%B7%E7%9A%84%E8%8F%9C%E9%B8%A1%E4%B9%9F%E5%8F%AA%E8%8A%B1%E4%BA%86%E4%B8%8D%E5%88%B0%E5%8D%8A%E5%B0%8F%E6%97%B6%E5%B0%B1%E5%AE%8C%E6%88%90%E4%BA%86%E7%A0%B4%E8%A7%A3%EF%BC%8C%E4%BD%95%E5%86%B5%E4%B8%93%E4%B8%9A%E7%9A%84%E9%80%86%E5%90%91%E5%A4%A7%E7%A5%9E%E5%91%A2%E3%80%82) ，[具体可参考2](https://blog.csdn.net/qq_41832837/article/details/113120047)

