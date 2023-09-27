# 移动端真机：调试H5或者app内嵌套的H5页面

通常情况下安卓浏览器或H5或app内嵌的H5都是给予`webview`来做的，最好用的工具当然还是 `chrome浏览器` 自带的开发者工具，不仅有Dom树，JS调试，网络监视等功能。

## 调试Android上`WebView`的步骤

1. 开启手机上的USB调试功能

2. 打开Chrome浏览器，地址栏输入：`Chrome://inspect`，回车

3. Chrome会自动检测手机上打开的App，并列出可调试的WebView页面，如图：

   ![image-20230927142109483](http://biji.51automate.cn/blogs/imgimage-20230927142109483.png)



### 发现问题

这里默认使用的是Chrome浏览器

#### 问题1、不显示待调试页面

实际使用过程中，Chrome中`chrome://Inspect` 无法识别、无法显示待调试页面

1. 反应慢，稍等一会

2. 重新打开USB调试开关

3. 打开USB调试和**仅充电模式下允许ADB调试。**

4. 只连接一个手机，拔掉其它的（不要同时连接多台设备）

5. 如果 `手机型号` 成功识别到了，但是没有识别到已打开的`WebView`。

   > 很**可能是要调试的APP没有打开WebView的调试模式**（不打开是没办法识别的）

   - 比如我想调试：微信内打开的H5(webview)页面,但是识别不到微信，当然不可以识别到微信内的页面

     微信升级后不再使用x5内核，原本使用`debugx5.qq.com`的方法就打不开了，无法打开vConsole查看页面请求等信息。

     有没有什么方法：开启微信调试？

     ```
     http://debugxweb.qq.com/?inspector=true
     ```

     1. 手机用USB连接至电脑（注意选择传输文件/调试模式，且手机需要开启uSb调试）

     2. 手机微信内点击以下网址

        ```
        http://debugxweb.qq.com/?inspector=true
        ```

        发现会跳转到微信首页，调试模式就开启了,然后你自行关闭即可

     3. 微信内打开所需调试网址

     4. chrome浏览器 打开`chrome:/inspect/ `就会看到WebView in com.tencent.mm。该项目下就是微信内我们打开的页面

        ![image-20230927143258619](http://biji.51automate.cn/blogs/imgimage-20230927143258619.png)

     5. 点击`inspect` 即可进入调试（调试需开启科学上网工具，否则会404）

   - 比如我想调试：游戏APP的内Webview页面，但是打包的时候调试模式是关闭的。
     其实大部分App的debug模式都是关闭的。

     解决方案：

     要么让`客户端同学打包个debug版本`（实用）

     要么借助第三方工具来强制开启任何App的Android webview debug模式，就要使用Xposed框架。有点麻烦，具体怎么用有时间了再总结吧。

     

#### 问题2、`Inspect`出现404页面

![image-20230927143649929](http://biji.51automate.cn/blogs/imgimage-20230927143649929.png)

原因：国内网络限制

> Inspect调试需开启**科学上网**工具，否则出现空白或404页面

解决方案：

> vpn翻墙



##### 没有翻墙工具怎么办？

1. 使用edge

   ```
   操作步骤
   1. 下载Edge浏览器
   2. 打开edge://inspect/#devices
   3. 点击inspect即可
   ```

   ![image-20230927144048508](http://biji.51automate.cn/blogs/imgimage-20230927144048508.png)

2. 使用QQ浏览器

   ```
   操作步骤
   1. 下载Edge浏览器
   2. 打开edge://inspect/#devices
   3. 点击 inspect fallback
   注意：点击的不是inspect
   ```

   ![image-20230927143116387](http://biji.51automate.cn/blogs/imgimage-20230927143116387.png)

   

## 调试IOS上`WebView`的步骤

打开手机点击设置 -> Safari浏览器 -> 高级

打开`JavaScript和web检查器`

![image-20230927162638311](http://biji.51automate.cn/blogs/imgimage-20230927162638311.png)

打开手机上的Safari，输入一个网址[www.baidu.com](https://link.juejin.cn/?target=http%3A%2F%2Fwww.baidu.com)

在Mac上打开Safari，点击开发-iPhone lh-[www.baidu.com，](https://link.juejin.cn/?target=http%3A%2F%2Fwww.baidu.com%EF%BC%8C) 就可以看到以下页面

![image-20230927162622526](http://biji.51automate.cn/blogs/imgimage-20230927162622526.png)





### 朋友推荐：全平台调试工具 

# Mobile Debug

[`Mobile Debug`官网下载地址](https://link.juejin.cn/?target=https%3A%2F%2Fwww.mobiledebug.com%2F)



怎么判断是否是H5页面？

H5页面白屏可能的原因？