# MyHexoBlogs-使用说明

### 创建新文章

将写好的文章复制到博客根目录 `myblogs`文件夹下` source/_posts` 里面即可

### 打包发布
#### 1、检查Nginx服务是否开启

```bash
# 云服务器执行 如下命令。如果返回结果的话，说明有nginx在运行，服务已经启动
ps -A | grep nginx

# 进入Nginx启动目录
cd /usr/local/nginx/sbin/
# 开启Nginx服务
./nginx -t
```



#### 2、将所有文章索引上传到algolia（全局搜索）
```bash
# 博客根目录执行
hexo algolia
```

#### 3、编译并部署到服务器

```bash
# 清理资源并重新编译运行
# hexo clear && hexo generate
hexo cl  && hexo g

# 部署到服务器
# dexo deploy
hexo d
# 新电脑没有权限推送到服务器，就需要输入账号密码 
# 服务器上的 git账户：git   密码： sXXXXXXX00

```

#### 4、刷新页面、查看效果

http://www.51automate.cn/



### 进阶使用

####  使用hexo,butterfly添加系列文章功能（二级目录）
当我们写一个系列文章的时候，想要在所属该系列的文章上方都展示一个类似于目录的结构，便于反复横跳整个系列。 效果如下
![](http://biji.51automate.cn/blogs/img/20230112213926.png)

**统一添加文章头部格式**

找到 `MyHexoBlogs\myblogs\scaffolds\post.md`  修改如下代码。

不想每次手动添加series的，通过命令`hexo new 文章名`就会自动加上上述标签和目录分类以及时间、作者信息、关键字、文章描述、文章缩略图，更重要的是 **series 系列**，比如 series : Python基础知识目录

```python
---
title: {{ title }}
date: {{ date }}
series:Python基础知识目录
tags:
    - test
calegories:
    - Tools
author: xuefeng365
keywords:
description:
avatar:/images/favicon.png
---
```

![](http://biji.51automate.cn/blogs/img/20230112202001.png)



**核心步骤，遍历展示目录结构**，

找到 `MyHexoBlogs\myblogs\node_modules\hexo-theme-butterfly\layout\post.pug`，新增一段代码如下

```python
extends includes/layout.pug

block content
  #post
    if top_img === false
      include includes/header/post-info.pug
    //- 新增开始
    if page.series
      div.post-series
        h3 #{page.series}-系列文章:
        - let list = site.posts.sort('date', -1)
        - list.each(function(article){
          if article.series == page.series
            - let link = article.link || article.path
            - let title = article.title || _p('no_title')
            li
                a.title(href=url_for(link) title=title)= title
        - })
    //- 新增结束
    article#article-container.post-content!=page.content
    include includes/post/post-copyright.pug
```



### 异常场景

1、**hexo deploy时出现的警告：LF will be replaced by CRLF**
原因：windows下换行符为CRLF，Linux下换行符为LF（使用Git命令行Git Bash，实际上就是相当于在linux环境），而git工作区默认换行符为CRLF，当执行git add ... 时就会出现警告！当最终push到远程仓库时git会统一格式全部转化为用CRLF作为换行符，故不必对其做转换操作，文本文件保持原样，只需执行以下命令即可。

解决方案

```python
git config --global core.autocrlf false //禁用自动转换
```

![](http://biji.51automate.cn/blogs/img/20230112201009.png)

备注：

1. `LF`：Line Feed 换行
2.  `CRLF`：Carriage Return Line Feed 回车换行键
3. windows下：CRLF（表示句尾使用回车换行两个字符，即windows下的"\r\n"换行）
4. unix下：LF（表示句尾，只使用换行'\n'）
5. mac下：CR（表示只使用回车'\r'）



2、`hexo d`上传是报错 ： `YAMLException: can not read a block mapping entry; a multiline key may not...`

![](http://biji.51automate.cn/blogs/img/20230112204259.png)











