# 爬虫系列：BeautifulSoup4解析

官网：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/


## 进阶

### find模糊搜索

```
import requests,re
from bs4 import BeautifulSoup

'''
    find()方法比较low  只存在网上祖传代码  个人不建议使用
    推荐后面的CSS选择器用法

	bs4官方文档-find模糊搜索：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id28
'''

html = requests.get('https://www.baidu.com')
html.encoding = 'utf-8'

soup = BeautifulSoup(html.text,'lxml')

'''
    recursive参数：如果只想搜索Tag的直接子节点，可以使用参数recursive = False
    find()---返回str
    find_all()---返回list  
'''
# print(soup.find(recursive=True))
# print(soup.find_all(recursive=True))



'''
    name参数：
    写法一：str类型，标签的名称，比如a,p,h1等
    写法二：可迭代对象，比如：['a','p']  ('a','p')
    写法三：正则表达式

    find_all()---返回list
'''
# print(soup.find_all(name='a'))
# print(soup.find_all(name=['a','p']))
# print(soup.find_all(name=re.compile('^a')))



'''
    limit参数：限制返回条数，比如：1，5，10

    find_all()---返回list
'''
# print(soup.find_all(name='a',limit=2))



'''
    attrs参数：根据指定属性搜索

    find_all()---返回list
'''
# attrs = {
#     'class':'mnav'
# }
# print(soup.find_all(attrs=attrs))



'''
    **keyward参数：任意参数映射

    注意：如果搜索class要写成class_  因为class是python关键字

    find_all()---返回list
'''
# print(soup.find_all(class_='mnav'))
# print(soup.find_all(id='kw'))



'''
    其他方法，官方都用详细介绍，这里就不演示了
    find(name,attrs,recursive,text,**kwargs)
    find_parents()
    find_parent()
    find_next_siblings()
    find_next_sibling()
    find_previous_siblings()
    find_previous_sibling()
    find_all_next()
    find_next()
    find_all_previous()
    find_previous()
'''



```





### CSS选择器

```
from bs4 import BeautifulSoup

'''
    select_one() 返回第一个节点 str类型
    select() 返回list类型

	bs4官方文档-CSS选择器：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id42
'''

HTML = '''
<!DOCTYPE html>
<html lang = "en">
<head>
	<title>123</title>

    <a id='one' href='https://example.com/demo1'>
        <p class='test'>Test1</p>
        <p class='test'>Test2</p>
    </a>
    <a id='two' href='https://example.com/demo2'>
        <p class='test'>Test3</p>
        <p class='test'>Test4</p>
    </a>
    <a id='three' href='https://example.com/demo3'>
        <p class='test'>Test5</p>
        <p class='test'>Test6</p>
    </a>
	
    
    <a class="h">
        <p>干扰项标签</p>
    </a>

    <a class="h hh hhh hhhh">
        <p>正确标签</p>
    </a>

</head>
<body>
</body>
</html>
'''

soup = BeautifulSoup(HTML,'lxml')


'''
    按照标签名搜索
'''
# print(soup.select_one('a'))
# print(soup.select('a'))



'''
    获取该节点属性
'''
# print(soup.select_one('a')['href'])



'''
    获取该节点的内容
'''
# print(soup.select_one('a').text)



'''
    多个节点选择一个节点
'''
# print(soup.select('a')[0])



'''
    按照某个属性搜索（用 [] 声明）
    类似于正则表达式（支持 ^ $ *） 
'''
# print(soup.select("a[href='https://example.com/demo1']"))
# print(soup.select("a[href^='https://example.com']")) 
# print(soup.select("a[href$='demo1']"))
# print(soup.select("a[href*='demo']"))

soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]


'''
    按照id搜索（用 # 声明）
    id存在唯一性，使用select_one()即可
'''
# print(soup.select_one('a#one'))



'''
    按照class属性搜索（用 . 声明）
    class属性有可能存在多个标签，建议使用select()
'''
print(soup.select('p.test'))



'''
    按照标签名搜索---逐层搜索（用空格声明）
    类似于前后节点（不考虑节点关系） 
'''
# print(soup.select_one('a p'))
# print(soup.select('a p'))



'''
    按照标签名搜索---直接子节点（用 > 声明）
    类似于父子节点（考虑节点关系）
'''
# print(soup.select_one('a>p'))
# print(soup.select('a>p'))



'''
    按照标签名搜索---兄弟节点（用 ~ 声明）
    类似于兄弟节点（考虑节点关系）
'''
# print(soup.select_one('p.test~p'))
# print(soup.select('p.test~p'))



'''
    支持链式调用
'''
# print(soup.select_one('a').select('p'))



'''
    按照class属性搜索（用 . 声明）

    如果class属性有多个值，使用空格隔开，需要用 . 进行填充（参考链式调用）
'''
# print(soup.select_one('a.h>p'))             # 匹配失败  存在干扰项
# print(soup.select_one('a.h hh hhh hhhh>p')) # 匹配失败  无法找到标签
# print(soup.select_one('a.h.hh.hhh.hhhh>p')) # 匹配成功



'''
    常用混合写法：
    select()     选择父节点
    select_one() 选择子节点
'''
# for element in soup.select('a'):
#     print(element.select_one('p').text)
```



## 基础

### 文档树内容

```
from bs4 import BeautifulSoup

'''
	bs4官方文档-string：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#string
'''


HTML = '''
<!DOCTYPE html>
<html lang = "en">
<head>

	<title>Title</title>
	<a><p>Test</p></a>

</head>
</html>
'''

soup = BeautifulSoup(HTML,'lxml')


'''
    Tag包含多个节点，如果没有指定某个节点，string属性会是None

    a标签没有string
    p标签有string
'''
# print(soup.a.string)
# print(soup.p.string)




'''
    注意！！！
    空格和换行都算一个节点！！！

    <a><p>Test</p></a>

    以上格式，能正常输出


    <a>
        <p>Test</p>
    </a>

    以上格式，bs4会认为是3个节点
'''
# print(soup.a.string)





'''
    Tag包含多个节点，可以选择遍历获取全部string
    strings属性---返回生成器
'''
# for string in soup.head.strings:
#     print(string)



'''
    【此方法能去除空格、空行】
    Tag包含多个节点，可以选择遍历获取全部string
    stripped_strings属性---返回生成器
'''
# for string in soup.head.stripped_strings:
#     print(string)

```



### 父子节点

```
from bs4 import BeautifulSoup

'''
	bs4官方文档-父节点：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id24
    bs4官方文档-子节点：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id20
'''

HTML = '''
<!DOCTYPE html>
<html lang = "en">
<head>
	<title>123</title>
	<a><p>Test1</p><p>Test2</p></a>
</head>
<body>
</body>
</html>
'''

soup = BeautifulSoup(HTML,'lxml')  

'''
    输出该标签所有子节点---返回list
'''
# print(soup.a.contents)



'''
    输出某个标签的直接子节点---返回迭代器
'''
# print(soup.a.children)



'''
    遍历某个标签的直接子节点迭代器
'''
# for child in soup.a.children:
# 	print(child)



'''
    输出某个标签的所有子孙节点
'''
# for child in soup.descendants:
# 	print(child)



'''
    #获取当前Tag的父节点
'''
# print(soup.p.parent.name)



'''
    获取当前Tag所有的父辈节点
'''
# for parent in soup.p.parents:
# 	print(parent.name)



'''
    文本的父节点就是它自己
'''
content = soup.head.title.string
print(content.parent.name)
```



## 实战案例

```
待补充
```

