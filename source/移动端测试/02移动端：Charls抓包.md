# 移动端：Charls抓包

该软件是用Java写的，能够在Windows，Mac，Linux上使用。安装Charles的时候要先装好Java环境。

### 安装、注册

阿里云盘：搜索 charls 

![image-20230925145330512](http://biji.51automate.cn/blogs/imgimage-20230925145330512.png)

### 配置抓包信息

1. 代理和端口设置

   ![image-20230918092522944](http://biji.51automate.cn/blogs/imgimage-20230918092522944.png)

   ![image-20230918092559733](http://biji.51automate.cn/blogs/imgimage-20230918092559733.png)

2. 安装charls根证书

   ![image-20230925164826111](http://biji.51automate.cn/blogs/imgimage-20230925164826111.png)

   ![image-20230925164948596](http://biji.51automate.cn/blogs/imgimage-20230925164948596.png)

   













查看本机ip

1. ipconfig

2. 通过Charls自身查看

   ![image-20230918092642909](http://biji.51automate.cn/blogs/imgimage-20230918092642909.png)

   ![image-20230918093353002](http://biji.51automate.cn/blogs/imgimage-20230918093353002.png)

3. 打开手机->找到wifi->长按点击修改网络->点击代理->选择手动->配置服
   务器主机+端口->点击保存

   > 可参考 fiddler教程

4. 安装证书，（如果待抓包软件支持模拟器，推荐优先使用模拟器抓包 ）这里后给出模拟器和真机两种安装证书的方法

   > 可参考 fiddler教程

   真机：华为手机为例

   手机访问：`chls.pro/ssl`

   ![image-20230918093610902](http://biji.51automate.cn/blogs/imgimage-20230918093610902.png)

5. 安装好之后，一旦有网络请求，charls就会监听到，【弹框提示是否允许或拒绝】连接，一定要点击 允许，才能进一步抓包

   如果手抖，拒绝了，将无法抓包，弹窗也不再弹出了，怎么办？请执行下面一步

6. 在Charles的`代理`的`访问控制设置`中，把手机ip全部清除：

   ![image-20230918095725502](http://biji.51automate.cn/blogs/imgimage-20230918095725502.png)

   ![image-20230918095730865](http://biji.51automate.cn/blogs/imgimage-20230918095730865.png)

   


**以上是基础使用，其它实用技巧，以后在补充**
