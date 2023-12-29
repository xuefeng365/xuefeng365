[参考教程1](https://mp.weixin.qq.com/s/MzuBh7BcuTDYWyv0C8Z2DQ)

## Sphinx+gitee+Read the Docs搭建在线文档系统

### 安装

#### 安装环境

windows+python3.XX

#### Sphinx安装

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple sphinx
```

#### 项目初始化

新建一个文件夹`myblog`，进入该文件夹，执行cmd命令

> 然后会有如下的输出，需要根据提示输入项目名称、作者、版本号、语言等信息

```
# 初始化
sphinx-quickstart

# 选择语言
zh_CN
```

![image-20230815144039352](http://biji.51automate.cn/blogs/imgimage-20230815144039352.png)

初始化目录结构如下：

![image-20230817165622852](http://biji.51automate.cn/blogs/imgimage-20230817165622852.png)

![image-20230817175146986](http://biji.51automate.cn/blogs/imgimage-20230817175146986.png)

各个文件的作用：

- `build`：生成的文件的输出目录

- `source`: 存放文档源文件

- - `_static`：静态文件目录，比如图片等
  - `_templates`：模板目录
  - `conf.py`：进行 Sphinx 的配置，如主题配置等
  - `index.rst`：文档项目起始文件，用于配置文档的显示结构

- `cmd.bat`：这是自己加的脚本文件（里面的内容是‘cmd.exe’）,用于快捷的打开windows的命令行

- `make.bat`：Windows 命令行中编译用的脚本

- `Makefile`：编译脚本，make 命令编译时用

### 构建编译

```
# 安装 autobuild 工具, 自动创建HTTP服务，可以在浏览器器中通过ip地址来查看 

pip install -i https://pypi.tuna.tsinghua.edu.cn/simple sphinx-autobuild

# 编译命令
sphinx-autobuild source build/html
```

![image-20230815142848392](http://biji.51automate.cn/blogs/imgimage-20230815142848392.png)

### 更改样式主题

默认的主题`alabaster`，如果想安装其它的主题，可以先到Sphinx的官网https://sphinx-themes.org/查看，多数人使用这个`Read the Docs`主题。

```
# 安装主题
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple sphinx_rtd_theme
```

然后修改`conf.py` 文件，找到 `html_theme` 字段，修改为

```
# html_theme = 'alabaster'
html_theme = 'sphinx_rtd_theme'
```

安装markdown支持工具

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple recommonmark
```

若要使用markdown的表格，还要安装：

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple sphinx_markdown_tables
```

注：支持markdown后，文档文件可以使用markdown格式，但文档的配置文件`index.rst`还要使用reST格式

### 设计博文的显示结构

#### index.rst

修改文档结构，需要修改`index.rst`文件，首先来看一下这个文件中的内容：

```
.. demo1 documentation master file, created by
   sphinx-quickstart on Tue Aug 15 14:41:19 2023.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to demo1's documentation!
=================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:



```



- 两个点`.. `+空格+后面的文本，代表注释（网页上不显示）
- 等号线`====`+上一行的文本，代表一级标题
- `.. toctree::`声明的一个树状结构（Table of Content Tree）
- `:maxdepth: 2` 表示页面的级数最多显示两级
- `:caption: ` 用于指定标题文本（可以不要）
- 最下面的3行是索引和搜索链接（可以先不用管）

#### 修改index文件

修改soure文件夹下的index.rst文件,，这里表示添加了一个Cpp目录，然后Cpp目录下，链接的又一个index文件

```
Welcome to demo1's documentation!
=================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

```



### 遇到的问题

#### 关于Gitee的仓库

> 一定**公开**的 ， 不能是私有的。

#### 关于分支 

> gite 默认分支是master作为主分支
>
> github 默认分支是main   作为主分支
>
> 这里使用的是gitee 
>
> 请注意保持一致

![image-20230816191242406](http://biji.51automate.cn/blogs/imgimage-20230816191242406.png)

#### 关于构建时 缺少插件

大概率是版本问题，因为`Read the Docs` 默认使用的是 Python3.xx的版本, 你可以去项目高级设置里看到。

![image-20230816192127662](http://biji.51automate.cn/blogs/imgimage-20230816192127662.png)

解决方案：添加一个 `requirements.txt` 指定插件的版本号。

教程如下：https://blog.csdn.net/wenbo13579/article/details/126153208



#### 自动构建--webhooks工具

`gitee` 的 `webhook` 请求发送的格式与 `readthedocs` 所需要求的格式不匹配，经查阅 `readthedocs` 官方文档，可以手工发送 `webhook` 请求，具体方法[可参考](https://blog.csdn.net/icbm/article/details/81265908)



适用方向：

1. jenkens：代码更新自动打包

   > https://blog.csdn.net/m0_46267097/article/details/107487615

2. 博客自动构建, 分为 Gitee和Github

3. 

以下重点说一下gitee 自动构建博客：设置钩子，远程仓库更新后，自动构建最新博客。github比较简单，可参考官网教程

将文档托管到gitee上面，push源码后自动构建发布到`read the docs`上面， 这样既有版本控制好处，又能自动发布到`read the docs`。

gitee 官网提供的[webhooks操作教程](https://help.gitee.com/webhook/how-to-add-webhook/)



需要注意的是在Build version之前， 需要先添加集成，再在gitee项目里添加WebHook.



`read the docs`设置如图![image-20230818095339068](http://biji.51automate.cn/blogs/imgimage-20230818095339068.png)

![image-20230928160658778](http://biji.51automate.cn/blogs/imgimage-20230928160658778.png)

![image-20230928160741308](http://biji.51automate.cn/blogs/imgimage-20230928160741308.png)

[如何手动配置 Git 存储库集成 — 阅读文档用户文档 (docs.readthedocs.io)](https://docs.readthedocs.io/en/stable/guides/setup/git-repo-manual.html#webhook-integration-generic)

![image-20230928190833367](http://biji.51automate.cn/blogs/imgimage-20230928190833367.png)

#### 修改博客显示的、短链接

想要通过自己的子域名进行解析。要配置此域，请在DNS中添加CNAME记录，将自定义域指向`readthedocs.io`。

xuefeng365.gitee.io/blog

如何将 Gitee 作为自己的二级域名

https://www.jianshu.com/p/a9a9d104aa5a



设置好域名后，再去域名服务器添加一条解析记录。静静的等待10分钟。

![image-20230821165615897](http://biji.51automate.cn/blogs/imgimage-20230821165615897.png)

域名服务器操作

![image-20230821165504726](http://biji.51automate.cn/blogs/imgimage-20230821165504726.png)

![image-20230821165449700](http://biji.51automate.cn/blogs/imgimage-20230821165449700.png)

![image-20230821173159522](http://biji.51automate.cn/blogs/imgimage-20230821173159522.png)

![image-20230822103740755](http://biji.51automate.cn/blogs/imgimage-20230822103740755.png)

##### 解析域名的时候遇到的问题

阿里注册的域名，之前做博客的时候，把DNS解析服务器修改成了 

```
horace.dnspod.net
laurel.dnspod.net
```

导致新添加的解析记录不生效，需要先修改为 阿里云自动分配的 云解析 服务器。

```
dns25.hichina.com
dns26.hichina.com
```

[修改链接](https://dc.console.aliyun.com/next/index?spm=a2c4g.97785.0.0.79e23b98v4Y53d#/domain-list/all)

![image-20230822093459095](http://biji.51automate.cn/blogs/imgimage-20230822093459095.png)

#### 插入图片说明

小乌龟进行版本管理（合并、解决冲突）



### 开代理出现的问题 

git提交或克隆报错

`fatal: unable to access 'https://github.com/xuefeng365/blog.git/': OpenSSL SSL_read: Connection was reset, errno 10054`

因为git在拉取或者提交项目时，中间会有git的http和https代理，但是我们本地环境本身就有SSL协议了，所以取消git的https代理即可，不行再取消http的代理。

后续
原因还有一个，当前代理网速过慢，所以偶尔会成功，偶尔失败。

解决方案
1.在项目文件夹的命令行窗口执行下面代码，然后再git commit 或git clone
取消git本身的https代理，使用自己本机的代理，如果没有的话，其实默认还是用git的

```
//取消http代理
git config --global --unset http.proxy
//取消https代理 
git config --global --unset https.proxy
```



2.科学上网（vpn）
这样就能提高服务器连接速度，能从根本解决 time out 443问题





**文档引用：**

```
自定义引用文字

:doc:`引用本地的其它rst文档,rst后缀要省略，如about_us <../about_us>`

使用标题文字
:doc:`../about_us`
```

![image-20230927194303258](http://biji.51automate.cn/blogs/imgimage-20230927194303258.png)

**插入超链接**

```
直接嵌入网址：
`野火公司官网 <http://www.embedfire.com>`_

使用引用的方式把具体网址定义在其它地方： `野火公司官网`_

.. _野火公司官网: http://www.embedfire.com
```

![image-20230928100622211](http://biji.51automate.cn/blogs/imgimage-20230928100622211.png)

