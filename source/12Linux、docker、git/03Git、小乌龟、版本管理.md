# Git、小乌龟、版本管理-待完善

## Git安装

### Git 常用命令

```
待补充
```

## 小乌龟安装

安装包下载-阿里云盘：搜索 小乌龟

### 使用小乌龟进行代码推送



## 版本管理遇到的问题及解决方案

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

   - 在密钥列表中添加此密钥。

     ![image-20230925093919909](http://biji.51automate.cn/blogs/imgimage-20230925093919909.png)

     ![image-20230925094046551](http://biji.51automate.cn/blogs/imgimage-20230925094046551.png)

3. 将新密钥（公钥）复制到 GitHub或码云中

   **提醒：私钥需要妥善保存。如果没有私钥，上传至 GitHub 的公钥等于作废。**



