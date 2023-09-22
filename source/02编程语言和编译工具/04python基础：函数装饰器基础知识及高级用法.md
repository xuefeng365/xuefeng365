# python基础：函数装饰器基础知识及高级用法

闭包 就是为了解决函数嵌套时，局部变量在赋值前引用的问题（不同作用域调用的问题）。除了使用global声明全局变量外，还可以使用nonlocal 声明自由变量，实现闭包。

在此之前我们先了解一下基础知识。

##  作用域

内建作用域（.py文件）、全局作用域、局部外的局部作用域（闭包函数外面-- 局部外的局部）、局部作用域

```
g_count = 0  # 全局作用域
def outer():
    o_count = 1  # 闭包函数外面的 局部作用域
    def inner():
        i_count = 2  # 局部作用域
```


## 全局变量和局部变量
内部作用域想修改外部作用域的变量时，使用 global 和 nonlocal 关键字了。

### 1.内部修改全局变量

```
# 修改全局变量 num
num = 10
print(f'内部修改全局变量前 num：{num}')
def fun1():
    global num  # 需要使用 global 关键字声明
    num = num + 1   # 内部修改全局变量
fun1()
print(f'内部修改全局变量后 num：{num}')
```



### 2.嵌套函数内层函数 修改 局部外的 局部变量 ，使用nonlocal 关键字

```
num = 10
def outer():
    num = 555  # 闭包函数外的函数中
    print(f'嵌套函数 内部修改 局部外的局部变量 前num:{num}')
    def inner():
        nonlocal num  # nonlocal关键字声明
        num = num + 1
    inner()
    print(f'嵌套函数 内部修改 局部外的局部变量 后num:{num}')

outer()
print(f'全局变量num的值：{num}')

```



![image-20230222172412079](http://biji.51automate.cn/blogs/img/image-20230222172412079.png)

稍微修改一下，去掉`nonlocal num  # nonlocal关键字声明`

```
# 内层嵌套函数 修改 局部外的局部（非全局作用域）变量则需要 nonlocal
num = 10
def outer():
    num = 555  # 闭包函数（inner）外面--局部外的局部变量
    print(f'嵌套函数 内部修改 局部外的局部变量 前num:{num}')
    def inner():
        num = num + 1
    inner()
    print(f'嵌套函数 内部修改 局部外的局部变量 后num:{num}')

outer()
print(f'全局变量num的值：{num}')
```



不使用`nonlocal`进行关键字声明，内嵌函数`inner` 内  `num = num + 1` 本来想要调用 outer中的 `num`的值，会报错 - 变量在赋值前引用。

`UnboundLocalError: local variable 'num' referenced before assignment`



nonlocal作用具体作用：**把变量`num`标记为自由变量，即使在函数中`为变量赋值`了，也仍然是自由变量。**

自由变量
> 自由变量是相对 **全局变量和局部变量** 而言的。
> 子作用域内(inner)可以取到父作用域内(outer)的变量，对于 子作用域（inner）而言，就是自由变量。


### 注意点

1. 对于列表、字典等可变类型来说，添加元素不是赋值，赋值 不会改变自有变量的身份。
2. 对于数字、字符串、元组等不可变类型以及**None**来说，赋值会 将其 改变为 局部变量。

```
# 情况一 举例
def funA():
    # 可变类型
    count = {}

    def hello(new_value):
        print(f'打印：{count}')  # 成功
        count[new_value] = new_value  # 赋值

    return hello
    
a = funA()
a(111)
    
    
# 情况2 举例
def funA():
    # 不可变类型
    count = 1  # 或者 count = None

    def hello(new_value):
        print(f'打印：{count}')  # 报错
        count = new_value  # 赋值

    return hello

```





## 被装饰函数是否有入参，如何设计装饰器？

### 被装饰函数 无入参

函数装饰器`funA()`是一个可调用对象，它的参数`fn`是被装饰函数（**无入参**）。

```
# funA 作为装饰器函数
def funA(fn):

    def hello():
        print("在 被装饰函数 <前> 执行")
        fn()  # 被装饰函数（装饰器的入参）
        print("在 被装饰函数 <后> 执行")
        return fn()

    return hello

@funA
def funB():
    print("执行被装饰函数")
    return '执行完毕'

a = funB()

```

程序执行流程为：

```
在 被装饰函数 <前> 执行
执行被装饰函数
在 被装饰函数 <后> 执行
执行完毕
```

在此基础上，如果在上面程序末尾 打印funB 函数的返回值

```
print(a)

# 其输出结果为
执行完毕
```

### 被装饰函数有入参，如何写装饰器

```
# *args 和 **kwargs 作为装饰器内部嵌套函数的参数，*args 和 **kwargs 表示接受任意数量和类型的参数

def funA(fn):
    # 定义一个嵌套函数（作用：获取函数的运行时间）
    def hello(*args, **kwargs):
        print(f"在 被装饰函数:{fn.__name__} <前> 执行")
        ret = fn(*args, **kwargs)
        print("在 被装饰函数 <后> 执行")
        return ret
        
    return hello

@funA
def funB(data1, data2):
    print(f"执行被装饰函数，入参:{data1}, {data2}")


funB("oh", "it is ok")
```

![image-20230224155908025](http://biji.51automate.cn/blogs/img/image-20230224155908025.png)

## 嵌套函数装饰器

```
@funA
@funB
@funC
def fun():
    #...
```

上面程序的执行顺序是里到外，所以它等效于下面这行代码

```
fun = funA( funB ( funC (fun) ) )
```

如果需要装饰器 对 带有函数的返回值进行修改，而不是直接返回被装饰函数的返回值 ，该怎么处理？ 

![image-20230224155042330](http://biji.51automate.cn/blogs/img/image-20230224155042330.png)



### 函数装饰器小结：

函数装饰器 funA() 去装饰另一个函数 funB()，其底层执行了如下 2 步操作：

1. 将 funB（） 作为参数传给funA（） 函数；

2. funA（）、funB（）接受相同的参数

3. 同时还可以做一些额外的操作

4. 返回被装饰函数funB（）本身的返回值

   

## 参数化装饰器

怎么让装饰器接受参数呢？答案是：创建一个装饰器工厂函数，把参数传给它，返回一个装饰器，然后再把它应用到要装饰的函数上。

待补充。。。。。。。。。



实际应用

我写过的几个装饰器。
