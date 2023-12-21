# Git、小乌龟、版本管理-待完善

## Git

### 下载安装

[教程](https://blog.csdn.net/Wang_XB_3434/article/details/131345012#t4)

[镜像](https://registry.npmmirror.com/binary.html?path=git-for-windows/)





打开cmd ，使用 `git -v` 命令，测试git环境变量是否已经添加；如果没有手动添加一下（我默认会安装到`D:\Program Files\Git`，环境变量path路径中 保证有这个路径）

> 

```
D:\Program Files\Git
```

![image-20231116101657213](http://biji.51automate.cn/blogs/img/image-20231116101657213.png)



### Git 常用命令

```
待补充
```

## 小乌龟

安装包下载-阿里云盘：搜索 小乌龟

### 流程演示

展示基本的 新建切换合并分支，日志信息和版本分支图使用、删除分支、解决简单冲突等。

新建空白项目`gametest`，雪峰和小明各自clone项目，各自管理。

![image-20231008160952987](http://biji.51automate.cn/blogs/imgimage-20231008160952987.png)

两个人各自新建一个自己的分支，分别负责开发一个新功能；

雪峰操作：

- 新增 `media`文件夹（包含 2个mp3格式文件）和 `panda.png`文件 并修改`.gitignore`文件（具体方法下面有介绍），然后提交并推送

  ![image-20231008162037886](http://biji.51automate.cn/blogs/imgimage-20231008162037886.png)

- 新建 `release`分支（10月10号发布）

- 新建分支 `dev-xf`，进行下面的开发

  ![image-20231008163217794](http://biji.51automate.cn/blogs/imgimage-20231008163217794.png)

  ![image-20231008163535757](http://biji.51automate.cn/blogs/imgimage-20231008163535757.png)

- 切换到该`dev-xf`分支

  ![image-20231008164348631](http://biji.51automate.cn/blogs/imgimage-20231008164348631.png)

  ![image-20231008164403084](http://biji.51automate.cn/blogs/imgimage-20231008164403084.png)

  ![image-20231008164803981](http://biji.51automate.cn/blogs/imgimage-20231008164803981.png)

  

- 修改`README.md`文件内容 ：（此刻是在`dev-xf`分支）,并提交推送

  ![image-20231008163814360](http://biji.51automate.cn/blogs/imgimage-20231008163814360.png)

  ![image-20231008171017837](http://biji.51automate.cn/blogs/imgimage-20231008171017837.png)

- 创建版本号标签 方便管理

  ![image-20231008171414349](http://biji.51automate.cn/blogs/imgimage-20231008171414349.png)

  ![image-20231008171614636](http://biji.51automate.cn/blogs/imgimage-20231008171614636.png)

- 新增 `xf01.md`,提交推送，并设置标签 `v1.0.2`

  ![image-20231008172348541](http://biji.51automate.cn/blogs/imgimage-20231008172348541.png)

  

小明操作：

- 修该了`master`分支中 `README.md`内容，并提交到仓库，导致雪峰本地的`master`内容不一致（这种拉取一下最新的内容即可）。同时雪峰的分支线 dev-xf中`README.md`

  ![image-20231008174529591](http://biji.51automate.cn/blogs/imgimage-20231008174529591.png)

雪峰操作：

- 切换到`master`分支

- 修改master中的 `README.en.md`内容并推送

  果然推送失败 。现在仓库中 `README.md`文件已经被小明推送了而雪峰的本地还是旧的， 尽管这次并没有该这个文件的内容，但是每次提交前都需要保证你的本地仓库是最新的。

  ![image-20231008180134701](http://biji.51automate.cn/blogs/imgimage-20231008180134701.png)

  当然拉取后，可以查看本地和仓库的差异。

  ![image-20231008180333041](http://biji.51automate.cn/blogs/imgimage-20231008180333041.png)

  ![image-20231008194434496](http://biji.51automate.cn/blogs/imgimage-20231008194434496.png)

- 请注意上面的拉取动作只是将远程仓库更新到了本地master分支，但是本地修改的 `README.en.md`内容还没有被成功推送（上一步失败了），所以还有执行一次推送操作。

小明操作：

- 新建分支 `dev-xm`，并切换到该分支，进行新功能开发

  ![image-20231008192922742](http://biji.51automate.cn/blogs/imgimage-20231008192922742.png)

- 新增 `xm01.md`文件，推送

  ![image-20231008193517676](http://biji.51automate.cn/blogs/imgimage-20231008193517676.png)

- 想要合并分支到master，先切到master分支，然后合并

  ![image-20231008194731242](http://biji.51automate.cn/blogs/imgimage-20231008194731242.png)

  ![image-20231008194000010](http://biji.51automate.cn/blogs/imgimage-20231008194000010.png)![image-20231008194130462](http://biji.51automate.cn/blogs/imgimage-20231008194130462.png)

- 当前小明的本地master分支中 `README.en.md`内容并不是最新的，因为雪峰在上一次的master分支提交中，修改过里面的内容。为了展示解决冲突的问题，我们先在小明的本地，也修改一下这个文件的内容

  ![image-20231008195326548](http://biji.51automate.cn/blogs/imgimage-20231008195326548.png)

- 删除分支

  ![image-20231008205557825](http://biji.51automate.cn/blogs/imgimage-20231008205557825.png)

- 提交推送

  果然后冲突，重新推送吧

  ![image-20231008203135936](http://biji.51automate.cn/blogs/imgimage-20231008203135936.png)

  ```
  可以选择某一行内容，右键 选择使用此文本块
  也可以自行选择使用左边还是右边，
  或者优先使用左边内容（左右都改了同一行内容，左边为准），其余内容会合并
  ```

  ![image-20231008201744419](http://biji.51automate.cn/blogs/imgimage-20231008201744419.png)

  ![image-20231008203037879](http://biji.51automate.cn/blogs/imgimage-20231008203037879.png)

  



#### 小知识 

1. Git中，origin / master与origin master之间有什么区别？

   当我们从远程仓库`clone`时，仓库服务器（远程主机）的默认名字就被定义为 `origin` ，

   `master`、`origin/master`是两个本地分支，`origin`一个远程主机名。

   本地分支

   - `master` git默认本地分支名

   远程分支名

   `origin/master`，是从远程拉取代码后，在本地建立的一个副本（因此也有人把它叫作本地分支），默认是不显示的，小乌龟可以看到。

   远程主机名

   - `origin` 是一个`远程git服务器的一个简称`或者叫`远程主机名`

   

2. 日志信息![image-20231008204820194](http://biji.51automate.cn/blogs/imgimage-20231008204820194.png)



### 版本回退（回滚）

待补充





### 忽略（gitignore）文件

作用：设置不想被提交到仓库的内容。

![image-20231008104656750](http://biji.51automate.cn/blogs/imgimage-20231008104656750.png)

使用方法：

1.1 新建文件，并修改拓展名为: `gitignore`

1.2 删除文件名只保留 `.gitignore`

1.3  `.gitignore`文件中输入要忽略的规则

```
# #号后面 书写备注信息

# 忽略文件夹下所有文件
/media
# 忽略所有 png 的文件
*.png
# 还可以设置反忽略 -- 不忽略pdf文件（用处不多，不做介绍）
!*.pdf
# 空文件夹会被自动忽略掉
```

![image-20231008105115834](http://biji.51automate.cn/blogs/imgimage-20231008105115834.png)![image-20231008105208476](http://biji.51automate.cn/blogs/imgimage-20231008105208476.png)

![image-20231008105347311](http://biji.51automate.cn/blogs/imgimage-20231008105347311.png)

![image-20231008105816084](http://biji.51automate.cn/blogs/imgimage-20231008105816084.png)



## github

### 搜索技巧

1. 项目源码 按 `s `（搜索的意思）
2. 输入神秘代码

### 查找文件

1. 在主页按 `t`：查找文件

2. 点进源代码后，按`L`：跳转到某一行

3. 点击行号：

   - 复制这行代码
   - 生成永久链接

   ![image-20231009103223728](http://biji.51automate.cn/blogs/imgimage-20231009103223728.png)

源代码中按`b`：查看文件的改动记录

完整效果如下图

![100000002](http://biji.51automate.cn/blogs/img100000002.gif)



### 阅读代码技巧

很多时候都是下载到本地，其实可以在线阅读。

在线vscode：仓库详情界面按下“。”键

![100000001](http://biji.51automate.cn/blogs/img100000001.gif)

### 在线运行项目

在项目地址前加上`gitpod.io/#/`前缀

https://github.com/liyupi/sql-mother 》》 https://gitpod.io/#/github.com/liyupi/sql-mother

https://github.com/youhuangla/Note/blob/main/web/img/image-20220504231259146.png)

## 版本管理遇到的问题及解决方案

#### 使用小乌龟clone的项目没有显示绿勾

解决方案：https://blog.csdn.net/XIA_1997/article/details/82801741





今天使用小乌龟 将本 Git 项目推送到 GitHub 时，GitHub 却一直报如下错误：

`You‘re using an RSA key with SHA-1, which is no longer allowed. Please use a newer client`

原来是 GitHub 在 2022 年 3 月 15 日之后将不再支持 `RSA` 算法生成的密钥，原因是 `RSA` 不够安全，而笔者之前一直是使用如下命令生成密钥对的：

`ssh-keygen -t rsa -C "邮箱"`

现在只要更改[加密算法](https://so.csdn.net/so/search?q=加密算法&spm=1001.2101.3001.7020)即可，可以选择 `ed25519`

### 解决方案

1. 使用git生成密钥对

   `ssh-keygen -t 加密算法 -C "邮箱"`

   

   ```
   ssh-keygen -t ed25519 -C "邮箱"
   ssh-keygen -t ed25519 -C "120158568@qq.com"
   
   # 注意 引号不能省略；加密算法使用 `ed25519`
   ```

   

   执行命令后，会在`C:\Users\Windows 用户名\.ssh`文件夹下面生成两个文件，![image-20230925092831418](http://biji.51automate.cn/blogs/imgimage-20230925092831418.png)

   其中第一个是私钥，第二个是公钥，公钥需要提供给github或者码云

2. 使用小乌龟生成密钥对

   - 运行小乌龟安卓目录的bin文件下的 PuTTYgen程序

     ![image-20230925093303755](http://biji.51automate.cn/blogs/imgimage-20230925093303755.png)

   - 选择加密算法，并生成密钥对

     ![image-20230925093632549](http://biji.51automate.cn/blogs/imgimage-20230925093632549.png)

   - 需要不断在 PuTTY 软件界面内晃动鼠标，因为生成密钥时需要鼠标移动来生成随机数。

     ![image-20230925093741319](http://biji.51automate.cn/blogs/imgimage-20230925093741319.png)

   - 生成成功之后，显示出来的就是公钥了。与前面在 Git 中一样，需要全文复制这个公钥，以及保存自己的私钥。

     ![image-20230925093839240](http://biji.51automate.cn/blogs/imgimage-20230925093839240.png)

   - 在密钥列表中添加此密钥（私钥）。

     运行在小乌龟安卓目录的bin文件下的 pageant 程序，

     ![image-20231116095739987](http://biji.51automate.cn/blogs/img/image-20231116095739987.png)
     
     找到右边下角图标
     
     ![image-20230925093919909](http://biji.51automate.cn/blogs/imgimage-20230925093919909.png)
     
     ![image-20230925094046551](http://biji.51automate.cn/blogs/imgimage-20230925094046551.png)

3. 将新密钥（公钥）复制到 GitHub或码云中

   **提醒：私钥需要妥善保存。如果没有私钥，上传至 GitHub 的公钥等于作废。**



