# 画图工具Excalidraw、drawio用法汇总

## Excalidraw
### 快捷键

- R → Rectangle，矩形
- D → Diamond，菱形
- O → Oval，椭圆
- A → Arrow，箭头
- T → Text，文字
- P → Pan，画笔
- Shift + H → Flip horizontal，水平翻转
- Shift + V → Flip vertical，垂直翻转

长按 Shift 再执行某项操作：

- Shift + 拖动元素 → 水平或纵向移动
- Shift + 画矩形 → 正方形
- Shift + 画菱形 → 正方形
- Shift + 画椭圆 → 正圆
- Shift + 拖动图形控制点 → 等比例缩放
- Shift + 画箭头/划线 → 特定角度箭头/直线
- Shift + 拖动直线控制点变为孤形 → 特定位置

长按 Alt 再执行某项操作：

- Option + 画矩形/菱形/椭圆 → 从中心缩放
- Option + 拖动矩形/菱形/椭圆控制点 → 从中心缩放
- Option + Shift + 拖动矩形/菱形/椭圆控制点 → 从中心等比缩放
- Option + 拖动图形 → 复制出同样图形并移动


## drawio

### 两种连接方式

用箭头连接图形时，会有两种连接点：**固定的连接点**和**浮动的连接点**，固定的连接点呈现绿色，浮动的连接点呈现蓝色。
![image.png](http://biji.51automate.cn/blogs/img/202504232010265.png)
移动图形位置，固定的连接点是锁死的，而浮动的连接点会随着图形相对位置的变化而变动，并且始终保持最短路径。
![draw-move.gif](http://biji.51automate.cn/blogs/img/202504232011476.gif)

连接图形时，从绿色的连接点向外拖拽，会得到一个固定的连接点，同样指向目标图形的绿色连接点，也会得到固定的连接点。
![draw-move - 副本.gif](http://biji.51automate.cn/blogs/img/202504232011956.gif)

从悬浮的蓝色箭头向外拖拽，会得到一个浮动的连接点。同样鼠标直接指向目标图形，当边框呈现蓝色阴影时，也会得到浮动的连接点。
![draw-blue-arrow.gif](http://biji.51automate.cn/blogs/img/202504232012422.gif)
当然，如果想让一端为浮动连接，另一端为固定连接，也是可以的，按照上述具体操作即可。当我们明白了这两种连接方式的区别，移动图形时，就不会为箭头的变化感到迷惑了。

### 容器与成组

容器是 Draw.io 中的一个重要的概念，它的作用是封装子流程。当流程图非常复杂时，可以将属于某个环节的流程进行封装，以此起到简化流程的作用。

![draw-blue-arrow.gif](http://biji.51automate.cn/blogs/img/202504232013053.gif)
选中**单个图形**，使用快捷键 Ctrl + G（Windows），会将图形转化为容器，容器的左上角会出现一个可折叠的标记。接着将其他图形拖入容器中，拖入时容器边框会有紫色的阴影提示。
选中**多个图形**时，使用快捷键 Ctrl + G，结果是将元素合并成组。
