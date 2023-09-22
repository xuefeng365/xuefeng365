# pycharm常用快捷键

- ctrl + shift + V          调出pycharm 历史剪切板

- ctrl + shift + U          大小写转换 

- ctrl + O                      重写父类方法

  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-重写父类方法ctrl+O.gif)

- shift + Enter             快速开启新的一行

- ctrl + shift + Enter   末尾自动补全（自动结束代码，行末自动添加冒号、括号等）

- ctrl + W                     扩大选区

  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-ctrl+W扩大选区.gif)

- ctrl + J                       查看已有的代码模板

  > 常用代码可以设置成模板。
  >
  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-设置-查看使用代码模板.gif)

  

- alt + J                         选择光标处的符号 

  > 再次按下，多选择下一个同样的符号,不断按下，不断多选,
  >
  > alt + shift + J 取消上一个匹配项
  >
  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-alt+J.gif)

- ctrl + Alt + T              选中代码块-快速包围代码

- ctrl + alt + M             快速封装为方法

  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-快速封装为方法-ctrl+alt+M.gif)

- ctrl + alt  +  L  当前文件格式化    ctrl + alt + shift + L  设置格式化范围

  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-快速格式化文件代码.gif)    

  

- ctrl +  P                     查看该函数入参

- alt + Q                       看代码必备（显示这行代码处于哪个函数、哪个类）

- ctrl + Q                      快速预览这个类或者函数说明

- ctrl + shift + i            快速预览这个类或者函数的代码逻辑



## 搜索

- ctrl + B                       选择函数名后，精确搜索一个函数在整个项目中被谁调用 

  > 在函数名上按下 `鼠标中健` ，抬起后 也可以触发搜索
  >
  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-精确搜索函数在项目被谁调用.gif)
  >
  > 
  >
  > Ctrl + B 另一种用法（进入被调用方法的内部，类似 Ctrl + 鼠标左键）
  >
  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-看代码必备-进入函数内部.gif)

- ctrl + N（全局搜索）  /  ctrl  + F（当前文件搜索）       

  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-内容搜索ctrl+F-ctrl+N.gif)   

- ctrl + E                    调出刚刚打开过的文件

  > ![](http://biji.51automate.cn/blogs/img/202211101456140.gif)

- aaa

- ctrl + G                    定位到指定行

  > ![](http://biji.51automate.cn/blogs/img/pycharm快捷键-跳转到指定行ctrl+G.gif)

## 替换

- ctrl + R                     当前文件替换          ctrl + shift + R         全局文件替换
- saaa

## 正则处理

- 浏览器中拷贝的 headers 是没有加引号的，但是我们 Python 中 Headers 是要以字典形式传参数的。

  > 使用pycharm 自带的正则处理数据
  >
  > ```
  > 1、选中数据
  > 2、
  > 第一行【查找】：  (.*?):(.*)
  > 第二行【替换】：  '$1':'$2',
  > 3、点击 .*  开启正则匹配 
  > 4、全部替换
  > ```
  >
  > 操作过程，如下图
  >
  > ![截图_20221110092954](http://biji.51automate.cn/blogs/img/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AE-%E6%AD%A3%E5%88%99%E5%A4%84%E7%90%86header%E5%A4%B4.gif)