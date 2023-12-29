# 七牛云图床开启HTTPS请求的两种方式

#### https请求下的http图片

因为自己一直用的是`七牛云`来管理博客图片，发现在`chrome`下用`https`访问[博客](https://blog.51automate.cn/)时看不到博客里的图片，而使用微信访问`https`博客却可以正常看到图片。`chrome`前端报错如下：

![img](http://biji.51automate.cn/blogs/img/202312291043830.png)

打开`chrome`开发者工具可以发现，我的`http`图片被自动转成了`https`。

查了一下原因 

> `https`网址下，不支持`http`协议的图片加载
>
> `Chrome`自动将地址栏的`http`请求转成了`https`, 但是`https`网址下，不支持`http`协议的图片加载
>
> 我博客里面的图片是通过七牛云进行存储，但是我在七牛云对象存储还没开启`https`访问，于是就报错了。接下来的问题就是**如何开启七牛云图床开启`HTTPS`请求**

#### 域名：源站域名 vs CDN加速

打开七牛云对象存储->域名管理可以看到，七牛云支持两种方式给对象存储加域名：CDN加速域名、源站域名。

![img](http://biji.51automate.cn/blogs/img/202312291045129.png)

大致意思就是，源站域名是指请求直接打到对象存储。CDN加速的话，请求会打到边缘节点（而非对象存储），当节点找不到这个图片时，才会请求对象存储。所以理论上CDN域名速度 > 源站域名速度，这也只是理论上而已。大致过程如下图：

![img](http://biji.51automate.cn/blogs/img/202312291045332.png)

空间管理页面：https://portal.qiniu.com/kodo/bucket

源站域名，在上述页面直接点击右侧“配置HTTPS”，上传SSL证书后就可以开启`https`访问了。

CDN加速域名，则需要点击域名标题进入到CDN域名管理页面，再配置SSL证书：

![image-20231229105546240](http://biji.51automate.cn/blogs/img/202312291055404.png)![image-20231229105629291](http://biji.51automate.cn/blogs/img/202312291056642.png)



配置SSL证书
无论哪种方式，开启`HTTPS`都需要申请一个`SSL证书`才行。

我的域名是管理在阿里云上的，所以我直接去[阿里云申请免费的SSL](https://yundun.console.aliyun.com/?spm=5176.12818093.ProductAndService--ali--widget-home-product-recent.dre2.409a16d0xv6VDP&p=cas#/certExtend/free)

你还没购买的话，可以点击立即购买，选择免费的证书费用是0元。

![img](http://biji.51automate.cn/blogs/img/202312291045927.png)

完事之后可以免费创建20个证书~~

创建证书的时候，一定要填对域名呀！！

之后把`Apache证书`下载到本地，因为我们要把他上传到七牛云上~

![img](http://biji.51automate.cn/blogs/img/202312291045212.png)

下载后解压可以得到三个文件：

```
➜  7450261_static.hijerry.cn_apache ll
total 24
-rw-rw-r--@ 1 jerry  staff   1.6K  3 19 14:18 7450261_static.hijerry.cn.key
-rw-rw-r--@ 1 jerry  staff   1.6K  3 19 14:18 7450261_static.hijerry.cn_chain.crt
-rw-rw-r--@ 1 jerry  staff   2.1K  3 19 14:18 7450261_static.hijerry.cn_public.crt

```

打开七牛云的上传证书页面：https://portal.qiniu.com/certificate/ssl#cert

点击上传原有证书，把解压的xxx.key和xxx.public.crt的内容复制到下面两个框框内

![img](http://biji.51automate.cn/blogs/img/202312291046543.png)

然后再去 源站域名/CDN加速域名 开启`HTTPS`，选中这个证书就可以了~

#### 费用对比

图床的费用有如下几个部分，[价格和优惠](https://portal.qiniu.com/financial/price?product=kodo)可以查到。

- 存储空间。0~10G 免费
- CDN回源流量。0~10G 免费
- 外出流量，**0~100TB 0.28元/GB**
- CDN外出HTTP流量。0~10G 免费
- CDN外出HTTPS流量。**0~100TB 0.28元/GB**

`HTTPS`方案有两种组合：

- 源站域名：存储空间+外出流量
- CDN加速域名：存储空间+CDN回源+CDN外出`HTTPS`

两种组合下，存储0~10G都是免费，而`HTTPS`流量都需要 **0.28元/GB**，焯焯焯

