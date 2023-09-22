---
title: 基于django的平台建设-前端相关
series: pycharm
tags: [aamt,测试]
calegories: [接口自动化]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---



![image-20230317181219544](http://biji.51automate.cn/blogs/img/image-20230317181219544.png)

### vue项目结构

`main.js` 主程序入口 

utils 通用的一些函数方法 



vue项目是一个单页面应用，整个项目只有一个html ，项目的前端代码都是放在这个html的body 的div里面 

![image-20230317152409616](http://biji.51automate.cn/blogs/img/image-20230317152409616.png)

![image-20230317152422328](http://biji.51automate.cn/blogs/img/image-20230317152422328.png)



实际各个模块的前端代码存放在 src的 views文件夹的 .vue文件里。

![image-20230317151250169](http://biji.51automate.cn/blogs/img/image-20230317151250169.png)



接下来 进行 路由配置（文件之间的关系）， 就可以实现 访问不同的url ,  跳转到不同的页面。（这里是vue2的项目）

### vue-router 安装和配置的步骤

- 安装 vue-router 包

  **vue-router** 需要单独导入 并没有 继承到vue里 。

- **创建路由模块**

  在 **src** 源代码目录下，新建 **router/index.js** 路由模块，并初始化如下的代码：

  ![image-20230317162328258](http://biji.51automate.cn/blogs/img/image-20230317162328258.png)

  将views目录下的vue组件文件 配置成对应的url路径。

  ![image-20230317165341158](http://biji.51automate.cn/blogs/img/image-20230317165341158.png)

- 导入并挂载路由模块

  在 src/**main.js** 入口文件中，导入并挂载路由模块。

  ![image-20230317162150332](http://biji.51automate.cn/blogs/img/image-20230317162150332.png)

- 声明**路由链接**和**占位符**

  路由配好了，组件想要显示出来，还要一步

  > 我们在App.vue根组件中 划一块位置 （相当于 装一个插槽）：<router-view></router-view>，意思是 有个位置给你，子组件才能显示出来，没有位置即便是可以url路径配置好了，可以访问，也显示不了。当然如果有需要，你还可以在子组件上继续安装 插槽。

  ![image-20230317154450943](http://biji.51automate.cn/blogs/img/image-20230317154450943.png)

  

#### 声明路由的**匹配规则**

在 src/router/index.js 路由模块中，通过 **routes 数组**声明路由的匹配规则。示例代码如下：

![image-20230317165858412](http://biji.51automate.cn/blogs/img/image-20230317165858412.png)

#### 路由拦截器 （鉴权）

拦截器实现的功能：有登录token 保持在当前页，没有就 跳转到登录页

![image-20230317171013030](http://biji.51automate.cn/blogs/img/image-20230317171013030.png)

更多路由设置 参考 https://www.cnblogs.com/xydchen/p/16155505.html

### vue的全局配置

vue.config.js  vue的全局配置 ，比如全局代理 url是 /api，就去接fastapi的后端服务 

![image-20230317171404557](http://biji.51automate.cn/blogs/img/image-20230317171404557.png)
