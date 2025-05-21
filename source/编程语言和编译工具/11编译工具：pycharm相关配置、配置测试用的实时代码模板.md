# pycharm相关配置、配置测试用的实时代码模板

##  插件


1. chinese(simplified)
2. json parser
3. rainbow brackets
4. translation  
5. key promoter
6. Translation
7. CodeGlance  显示代码缩略图插件 
8. Vue.js
9. Gitee
10. pydantic语法提示
11. CodeGeex 人工智能代码插件



## pycharm 定义的函数 添加注释时,不出现方法里的参数，返回值等注释

pycharm 定义的函数 添加注释时（输入三个“““回车），不出现方法里的参数，返回值等注释

解决方案：如图

![image-20230512114419777](http://biji.51automate.cn/blogs/img/image-20230512114419777.png)



## pycharm中为写好的方法添加快速添加类型提示



![动画](http://biji.51automate.cn/blogs/img/%E5%8A%A8%E7%94%BB.gif)

## 添加国内镜像源

![image-20231114094605675](http://biji.51automate.cn/blogs/img/202311140946759.png)



```
http://pypi.douban.com/simple/ 豆瓣
http://mirrors.aliyun.com/pypi/simple/ 阿里
http://pypi.hustunique.com/simple/ 华中理工大学
http://pypi.sdutlinux.org/simple/ 山东理工大学
http://pypi.mirrors.ustc.edu.cn/simple/ 中国科学技术大学
https://pypi.tuna.tsinghua.edu.cn/simple 清华

```



## 配置测试用的实时代码模板

1. 打个专业的招呼

   ```
   # -*- coding: utf-8 -*-
   
   # @Software: ${PRODUCT_NAME}
   # @File: ${NAME}.py
   # @Author: xuefeng365
   # @E-mail: 120158568@qq.com,
   # @Site: www.51automate.cn
   # @Time: ${MONTH_NAME_SHORT} ${DAY}, ${YEAR}
   # @Des: 说明（比如 基本配置文件）
   
   ```

   ![image-20230512095529759](http://biji.51automate.cn/blogs/img/image-20230512095529759.png)

2. 测试同学实际工作常用的实时模板 

   如何设置模板，如下图

   ![image-20230512101122471](http://biji.51automate.cn/blogs/img/image-20230512101122471.png) 

   - class 创建类 - 实时模板

     ```
     class $class$(HttpClient):
         """
         说明
         """
     
         def __init__(self, $args$):
             """
             $class$的初始化函数
             """
             pass
             $end$
     ```

   - 列表或者详情接口返回 - 实时模板：方便快速断言

     ```
     class Clazz:
         # $txt$
         a = jmespath.search("result.records[0].num", ret).replace(',', '')
         num = float(a) if a else 0
     
         # $txt$
         a = jmespath.search(f"result.records[?warehouseName=='{shop_name}'].id|[0]", ret)
         id = a if a else ''
     
         # $txt$
         a = jmespath.search("result.records[0].warehouseName[].id|[0]", ret)
         id = a if a else ''
     
         # $txt$
         a = jmespath.search(f"result.records[?(upcCode=='{upcCode}')&&(newOldState=='{types}')].id|[0]", ret)
         id = a if a else ''
         
         # 完整响应对象 ret: 上述接口的响应response, json格式
         ret = ret
     
     
     return Clazz
     
     ```

   - 断言 - 实时模板

     ```
     assert jmespath.search('code', ret) < 400,f'异常， $txt$ ,接口报错：\n{ret}\n'
     ```

   - 快速发起请求 - 实时模板

     ```
     method = 'post' or 'get'
     
     # 获取其他接口写入缓存的关联数据
     self.cache.get('key1')
     # 提取key2数据，存入缓存，供其他接口使用
     self.cache.put({'key2':10086})
     
     body = {"key5": "A","key6":"B"}
     
     
     url = 'system/xuefeng/aaaa'
     # system/xuefeng/aaaa?key5=A&key6=B
     url = self.get_full_url(url, etc=body, h=self.host)
     
     url = 'system/xuefeng/aaaa'
     # 接口 和 host拼接
     url = self.get_full_url(url, h=self.host)
     
     url = 'system/xuefeng/aaaa/{}'
     # system/xuefeng/aaaa/key5
     url = self.get_full_url(url, replace=('key5'), h=self.host)
     
     
     ret = self.send(url=url, method=method, body=body, body_type=self.json, x_token=self.token)
     ret = self.send(url=url, method=method, body=body, body_type=self.form_text, x_token=self.token)
     
     file_path = get_file_path(file_name='xiaoxin.png')
     ret = self.send(url=url, method=method, body=body, body_type=self.form_file, file_key='picFile',file_path=file_path, x_token=self.token)
     ret = self.send(url=url, method=method, body=body, body_type=self.binary, file_path=file_path, x_token=self.token)
     
     assert jmespath.search('code', ret) < 400,f'异常, 新增{XX} 接口报错：\n{ret}\n'
     
     if jmespath.search('code', ret) < 400:
         print(f'XX {name} 新增成功')
     elif jmespath.search('code', ret) == 500 and :
         assert '已存在' in ret['message'], f'异常, 重复新增 {AA} 功能异常，未友好提示，响应信息\n{ret}\n'
     else:
         assert False, f'异常，新增 {AA} 异常，响应信息\n{ret}\n'
     
     ```


3. 快速查看已设置的模板 ：` ctrl + J `
