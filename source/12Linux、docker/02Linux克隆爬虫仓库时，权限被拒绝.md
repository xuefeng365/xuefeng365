---
title: Linux系统克隆仓库时，权限被拒绝，如何处理
series: Linux
tags: [python基础,Linux]
calegories: [Linux]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---

我自建IP池镜像的时候，想把仓库克隆到Linux系统上，进而通过docker 构建出自己的镜像，

发现权限被拒绝

![image-20230215175439466](http://biji.51automate.cn/blogs/img/image-20230215175439466.png)

原因是原有linux 系统机器上没有生成过秘钥，

```
# 通过以下命令：查看 SSH公钥是否存在
cd ~/.ssh
```

生成公钥

```
ssh-keygen -t rsa -C "120158568@qq.com"

# 一路回车，最后你可以看到，在自己的目录下，有一个.ssh目录，说明成功了
```

![image-20230215175748670](http://biji.51automate.cn/blogs/img/image-20230215175748670.png)

```
# 验证是否添加成功，
ssh -T git@github.com
# 成功后，选好目录，再次克隆即可
```

![image-20230215180105174](http://biji.51automate.cn/blogs/img/image-20230215180105174.png)


