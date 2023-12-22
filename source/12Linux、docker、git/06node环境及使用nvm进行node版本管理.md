# node环境及使用nvm进行node版本管理

## 卸载

如果电脑已经下载了node需要先卸载node，再安装nvm，以避免冲突。

参考：https://blog.csdn.net/dgfdhgghd/article/details/123356831

关键点：卸载主程序、删除环境变量、**重启电脑**

## 安装 node.js

### 官网下载：

https://nodejs.org/en/about/previous-releases

![img](https://cdn.nlark.com/yuque/0/2022/png/25763438/1666701660625-14c6ccf0-f3dc-415c-a600-33bc39c13df7.png)

### 淘宝镜像（选择V12以上的长期维护版）

https://registry.npmmirror.com/binary.html?path=node/

![img](https://cdn.nlark.com/yuque/0/2022/png/25763438/1666702601931-74c6c3b1-e3df-4fdf-a97f-00e1143b847c.png)





### windows 使用nvm管理node版本

下载的别人vue项目，和本机的node版本不一致，同时又不能直接升级node，以免我的其它项目收到影响。
最好能有个版本管理工具，比如 nvmw（windows使用，mac电脑使用nvm）

画重点 请注意一定要用管理员身份安装



```
git config --global url."https://".insteadOf git:/

git config --global url.“https://”.insteadOf ssh://git@

git config --global http.sslVerify "false"

```



#### nvm教程 

##### 1. 安装nvm

![image-20231127205436662](http://biji.51automate.cn/blogs/img/202311272054180.png)

![image-20231127205617679](http://biji.51automate.cn/blogs/img/202311272056610.png)

**注意：如果电脑已经下载了node需要先卸载node，再安装nvm，以避免冲突。**

正常情况下nvm的环境变量和nodejs的环境变量（包括**用户变量和系统变量**）已经被配置
如果没有配置，可以安装图片配置

![image-20231127205655569](http://biji.51automate.cn/blogs/img/202311272056688.png)



##### 2. 配置文件中修改镜像（用于nvm快速安装不同node版本）

```
root: D:\nvm\nvm
path: D:\nvm\nodejs
arch:64
proxy: none
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

![image-20231127205759024](http://biji.51automate.cn/blogs/img/202311272057936.png)



常用命令

```
nvm root 查看nvm的安装目录(我习惯安装在 D:\nvm)

nvm list available  显示可以安装的所有node.js的版本
nvm list  显示所有已安装的node.js版本
nvm use   切换到指定的nodejs版本
nvm install   安装指定版本的node.js，例如：nvm install 10.24.1  /  nvm install 12.22.12    /  nvm install 14.21.3  
nvm uninstall   卸载指定版本的node.js，例如：nvm uninstall 12.22.12
nvm on    启用node.js版本管理
nvm off    禁用node.js版本管理(不卸载任何东西)
```

##### 3. 安装node

> node官网 查看可下载的 release版本 ：https://nodejs.org/en/about/previous-releases
>
> 目前选用：10.24.1、12.22.12（win7最大支持版本）、14.21.3 三个版本

```
nvm install 10.24.1
nvm install 12.22.12

# 选择生效的node版本
nvm use 12.22.12
```



使用nvm 控制node版本时会出现全局包丢失现象–原因是之前安装的全局包可能是跟着之前版本的node 走的，而不是跟着正在挂在全局。

所以我们想办法将node 和全局安装包分开，这样切换node版本就只是简单的切换当前node版本，全局包始终存在。

![image-20231127210826862](http://biji.51automate.cn/blogs/img/202311272108097.png)

D:\nvm\nodejs\node_modules 新建  node_cache、node_global 文件夹

为 node_global 新建一个**环境变量**

![image-20231127211035054](http://biji.51automate.cn/blogs/img/202311272110401.png)



npm修改全局路径, 并设置环境变量

```
修改npm的包的全局安装路径
npm config set prefix "D:\nvm\nodejs\node_modules\node_global"
修改npm的包的全局缓存位置
npm config set cache "D:\nvm\nodejs\node_modules\node_cache"

# 验证全局路径是否生效
npm prefix -g
# 验证全局缓存cashe路径是否生效
npm config get cache

修官方镜像为国内的淘宝镜像（用于npm快速安装各种包）
npm set registry https://registry.npm.taobao.org
验证国内镜像是否生效
npm config get registry

# 查看配置列表
npm config ls
# 查看配置列表全部信息
npm config ls -l
```



nvm安装详解，nvm控制node npm版本修改（windows环境）

https://blog.csdn.net/o_0ava0_o/article/details/116025980



### 项目安装依赖包出现的一些异常

npm install 出现npm WARN tar ENOENT: no such file or directory, ..../node_modules/.staging/...解决办法

```
删除package-lock.josn, 重新安装即可
```

#### 设置Github，使用https:// 来替换 git://

场景 当你想去克隆一个别人`github`上的repository时，发现系统不让你动，提示你防火墙禁止对git://的访问，这时候就只能用https://来访问repository

> 比如这里我安装![image-20231127204657173](http://biji.51automate.cn/blogs/img/202311272046359.png)

执行以下代码，改为使用 https访问。

```
git config --global url."https://".insteadOf git://
```

此时 你的.gitconfig（C:\Users\Administrator(用户名)目录下）中会多出一行参数设置

![image-20231127204443723](http://biji.51automate.cn/blogs/img/202311272047201.png)

再次npm 安装就ok 了。



另外提醒一下：本地的git环境 最好设置好了用户名和邮箱，以及配置了公钥在github或者gitee，以免出现不必要的麻烦。
