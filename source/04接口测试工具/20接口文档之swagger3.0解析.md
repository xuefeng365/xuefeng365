---
title: swagger3解析工具
series: 测试
tags: [aamt,测试]
calegories: [接口自动化]
author: xuefeng365
description: 这里是描述
comments: 是否显示评论（true/false）
cover: img/1.png
---



实际工作中遇到的现象：开发同学对接口进行了变动，一般都只是前后端进行沟通，测试同学很晚或者根本不知道发生了变动。

导致的问题

1. 提测时（提测标准之一：原有的自动化脚本跑通），运行自动化脚本出错，定位分析问题耗时会较长。
2. 用于回归测试的自动化脚本 **影响范围**和**修改范围**不能有效定位，不利于测试同学维护自动化脚本，回归测试的耗时较长



还有一个痛点：

以前团队在书写自动化脚本接口入参和响应时，会基于swagger去对比入参、出参字段的描述，更好的理解每个字段的意思和类型、取值范围、取值含义， 如果接口复制出入参的时候可以将描述信息 一起转换为 json , 就可以直接复制到pycharm作为requests的出入参，根据字段描述简单修修改改，就可以发送请求了，无疑速度将会快很多，达到了降本增效的目的。



基于以上原因，开发出这样一个工具。



工具简介：

1. 基于swagger3 解析出来api的入参和出参（字段和出入参数描述）

2. 按需或定时构建、自动对比，生成 可视化的html 。

3. 支持多服务配置，自动对比，并备份上一次的报告。

4. 通过邮件服务器将报告发送给相关人员。

   > 不同环境哪些接口变动测试同学可以很方便的知悉，并可以根据变动的接口，定位脚本的可能影响的功能范围以及需要同步修改脚本的范围，极大提高工作效率。



新老接口对比使用到了 `difflib`库中`difflib.HtmlDiff().make_file() `方法

项目源码：待更新



以下是开发工具的过程中写给自己的总结

1. 什么时候 备份对比报告的文件 （每次生成报告前，先判断reports目录下 存在diff报告文件,就执行删除备份操作，备份文件按时间区分，精确到时分秒，方便后期比对）。

   遇到的问题：

   > 如果有多个服务一起执行，仅第一个服务前执行备份报告动作即可，提高效率。

   所以需要知道何时执行第一个，何时执行最后一个。

   > ![image-20230511134953476](http://biji.51automate.cn/blogs/img/image-20230511134953476.png)

2. 什么时候 备份为解析的swagger源json文件

   > 一旦调用服务就会备份文件，如果备份文件已经存在，除非手动删除，否则将不会再次备份。

   后期考虑根据 生产的 版本号不同，自动判断备份不同版本的swagger 源json文件。

3. reports 文件下的`Compare_the_file_diff.html` 报告文件，如果存在就执行备份，并删除。

   那么解析完最后一个服务后，才会对比 apis目录下的所有 新老接口json文件，如果都没有变动，就不会生成 `Compare_the_file_diff.html` 文件，想要上次的变动版本，请直接查看最新（时间）一条备份报告文件

4. 关于降噪处理。`modules_info_xxxx.json` 我主动添加的 `pull_time` 请求swagger服务的时间, 新老对比时，要过滤掉。

   > fileHandle.read().splitlines() 读取文件时，过滤掉 指定行。 注意只能按行过滤，不能精确到某一个字段

   ![image-20230511171033026](http://biji.51automate.cn/blogs/img/image-20230511171033026.png)

5. 一次对比多个服务接口变动，每一次都会对比 将 `swaggerBackUp/apis`文件下所有文件和`swagger/apis`文件下所有文件进行对比，导致每一次对比会将前一个服务不同的地方重复记录，最终生成的报告文件也会重复记录 。

   > 解决方案 增加标记，只有执行完最后一个服务才批量对比文件。

   ![image-20230511134953476](http://biji.51automate.cn/blogs/img/image-20230511134953476.png)

   

6. 发送邮件 只要有其中一个服务数据变动就发邮件。如有必要邮件正文可直接展示html报告内容

   ![image-20230511170448928](http://biji.51automate.cn/blogs/img/image-20230511170448928.png)

Jenkins自动构建（步骤）  待更新





emailhelper.py

```

# -*- coding: utf-8 -*-

# @Software: PyCharm
# @File: emailhelper.py
# @Author: xuefeng365
# @E-mail: 120158568@qq.com,
# @Site: 
# @Time: 11月 24, 2022

import smtplib  # 加载smtplib模块
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

from email.utils import formataddr
from email.mime.application import MIMEApplication
import time
from datetime import datetime


class EmailHelper(object):

    def __init__(self, title, content, sys_sender, sys_pwd, receiver):
        '''
        :param title:
        :param content: 邮件信息
        :param sys_sender: 发件人邮箱
        :param sys_pwd: 发件人邮箱密码
        :param receiver: 收件人邮箱
        '''
        self.title = title  # 标题
        self.receiver = receiver  #（收件人）要发送的邮箱地址
        self.content = content  # 发送内容
        self.sys_sender = sys_sender  # 系统账户
        self.sys_pwd = sys_pwd  # 系统账户密码

    def send_office365(self, file_list=None):
        '''
        发送邮件
        :param file_list: 附件文件列表
        :return: bool
        '''
        try:
            # 创建一个带附件的实例
            msg = MIMEMultipart()
            # 发件人格式
            msg['From'] = formataddr(("xuefeng365", self.sys_sender))
            # 收件人格式 (server.sendmail 里传参时的收件人是list，msg['to'] 接收的变量值是字符串－－－即在邮件里显示的收信人信息。)
            msg['To'] = self.receiver
            # 邮件主题
            msg['Subject'] = self.title
            str_ = ''
            for num, sever_name_json in enumerate(self.content,start=1):
                str_ = str_ + '<br/>' + f'{num}: ' + sever_name_json + '<br/>'

            # # 正文可以讲HTML显示再正文里
            # with open(f"{file_list[0]}", "r", encoding='utf-8') as f:
            #     text = f.read()
            #
            # # html文件插入正文
            # self.content = f'''
            #                                <H2>您好!</H2>
            #                                <p>以下服务的接口发生变动，请查看!&nbsp <br/> {self.content}</p>
            #                                ''' + text
            # html文件插入正文
            self.content = f'''
                                                       <H2>您好!</H2>
                                                       <p>以下服务的接口发生变动，详情请查看附件!&nbsp <br/> {str_}</p>
                                                       '''

            part_html = MIMEText(self.content, "html", "utf-8")
            msg.attach(part_html)

            data_now = datetime.now().strftime('%Y%m%d')
            # 邮件正文内容
            msg.attach(MIMEText(data_now + self.content , 'html', 'utf-8'))

            # ----------  上传附件模块------------
            # 附件列表 附件是和python文件在同一目录，请根据实际情况，修改附件的路径。
            if file_list is None:
                file_list = []
            # 多个附件
            for file_name in file_list:
                # 构造附件
                xlsxpart = MIMEApplication(open(file_name, 'rb').read())
                # filename表示邮件中显示的附件名
                xlsxpart.add_header('Content-Disposition','attachment',filename = '%s'%file_name)
                msg.attach(xlsxpart)
            # ----------  上传附件模块------------


            # SMTP服务器
            server = smtplib.SMTP("smtp.office365.com", 587,timeout=10)

            # 注意：附件是和python文件在同一目录，请根据实际情况，修改附件的路径。
            # 阿里云服务器，从即日起，不再提供25端口邮件服务 。必须使用SSL加密465端口发信！
            # 所以上面的代码中，改成了SMTP_SSL，并使用了465端口。
            # server = smtplib.SMTP_SSL("smtp.163.com", 465, timeout=10)

            server.ehlo()  # 向邮箱发送SMTP 'ehlo' 命令
            server.starttls()

            # 登录账户
            server.login(self.sys_sender, self.sys_pwd)
            # 发送邮件 （多个收件人容易出错， 核心问题在于server.sendmail 中的 多个收件人必须是list["邮箱A","邮箱B","邮箱C"] ）
            server.sendmail(self.sys_sender, self.receiver.split(','), msg.as_string())

            # 退出账户
            server.quit()
            print('邮件发送成功')
            return True
        except Exception as e:
            print('邮件发送失败 : ', e)
            return False

    def send_qq(self,file_list=None):
        '''
                发送邮件
                :param file_list: 附件文件列表
                :return: bool
                '''
        try:
            # 创建一个带附件的实例
            msg = MIMEMultipart()
            # 发件人格式
            msg['From'] = formataddr(("苏雪峰", self.sys_sender))
            # 收件人格式
            msg['To'] = self.receiver
            # 邮件主题
            msg['Subject'] = self.title

            # 正文
            self.content = '''
                                           <H2>您好!</H2>
                                           <p>{}</p>
                                           '''.format(self.content)

            data_now = datetime.now().strftime('%Y%m%d')
            # 邮件正文内容
            msg.attach(MIMEText(self.content + data_now, 'html', 'utf-8'))

            # ----------  上传附件模块------------
            # 附件列表 附件是和python文件在同一目录，请根据实际情况，修改附件的路径。
            if file_list is None:
                file_list = []
            # 多个附件
            for file_name in file_list:
                print("file_name",file_name)
                # 构造附件
                xlsxpart = MIMEApplication(open(file_name, 'rb').read())
                # filename表示邮件中显示的附件名
                xlsxpart.add_header('Content-Disposition','attachment',filename = '%s'%file_name)
                msg.attach(xlsxpart)
            # ----------  上传附件模块------------


            # SMTP服务器
            server = smtplib.SMTP_SSL("smtp.qq.com", 465, timeout=10)

            server.ehlo()  # 向邮箱发送SMTP 'ehlo' 命令

            # 登录账户
            server.login(self.sys_sender, self.sys_pwd)
            # 发送邮件 （多个收件人容易出错， 核心问题在于server.sendmail 中的 多个收件人必须是list["邮箱A","邮箱B","邮箱C"] ）
            server.sendmail(self.sys_sender, self.receiver.split(','), msg.as_string())

            # 退出账户
            server.quit()
            print('邮件发送成功')
            return True
        except Exception as e:
            print('邮件发送失败 : ', e)
            return False

    def send_163(self,file_list=None):
        '''
                发送邮件
                :param file_list: 附件文件列表
                :return: bool
                '''

        try:
            # 创建一个带附件的实例
            msg = MIMEMultipart()
            # 发件人格式
            msg['From'] = formataddr(("苏雪峰", self.sys_sender))
            # 收件人格式
            msg['To'] = self.receiver
            # 邮件主题
            msg['Subject'] = self.title

            # 正文
            self.content = '''
                                           <H2>您好!</H2>
                                           <p>{}</p>
                                           '''.format(self.content)

            # 邮件正文内容
            msg.attach(MIMEText(self.content, 'html', 'utf-8'))

            # ----------  上传附件模块------------
            # 附件列表 附件是和python文件在同一目录，请根据实际情况，修改附件的路径。
            if file_list is None:
                file_list = []
            # 多个附件
            for file_name in file_list:
                print("file_name",file_name)
                # 构造附件
                xlsxpart = MIMEApplication(open(file_name, 'rb').read())
                # filename表示邮件中显示的附件名
                xlsxpart.add_header('Content-Disposition','attachment',filename = '%s'%file_name)
                msg.attach(xlsxpart)
            # ----------  上传附件模块------------

            # SMTP服务器
            server = smtplib.SMTP_SSL("smtp.163.com", 465, timeout=10)
            # 阿里云服务器，从即日起，不再提供25端口邮件服务 。必须使用SSL加密465端口发信！
            # 所以上面的代码中，改成了SMTP_SSL，并使用了465端口。
            # server = smtplib.SMTP_SSL("smtp.163.com", 465, timeout=10)

            server.ehlo()  # 向邮箱发送SMTP 'ehlo' 命令
            server.starttls()

            # 登录账户
            server.login(self.sys_sender, self.sys_pwd)
            # 发送邮件 （多个收件人容易出错， 核心问题在于server.sendmail 中的 多个收件人必须是list["邮箱A","邮箱B","邮箱C"] ）
            server.sendmail(self.sys_sender, self.receiver.split(','), msg.as_string())

            # 退出账户
            server.quit()
            print('邮件发送成功')
            return True
        except Exception as e:
            print('邮件发送失败 : ', e)
            return False

if __name__ == '__main__':
    # 收件地址
    receiver = "xuefeng@163.com,120158568@qq.com"
    # 标题
    title = "33测试告警"
    # 开始时间
    start_time = time.strftime('%Y-%m-%d %H:%M:%S')
    ip = "xx.xx.xx.xx"
    # 发送内容
    content = "{} ip: {} 掉线".format(start_time, ip)

    # 365邮箱
    ret = EmailHelper(title, content, 'xuefeng.su@AAAA.com', 'XXX', receiver).send_office365()
    # 网易邮箱
    # ret = EmailHelper(title, content, 'xx@aa.com', 'xxx', receiver).send_163()
    # # QQ邮箱
    # ret = EmailHelper(title, content, '120158568@qq.com', 'xxxxxxxxxxxx', receiver).send_qq()

```



笛卡尔积/pairwise算法

```
# -*- coding: utf-8 -*-

# @Software: PyCharm
# @File: Pairwise.py
# @Author: xuefeng365
# @E-mail: 120158568@qq.com,
# @Site: www.51automate.cn
# @Time: 5月 06, 2023
# @Des: parewise算法



import copy
import itertools
from sys import stdout

from loguru import logger


def parewise(option):
    """pairwise算法"""
    cp = []  # 笛卡尔积
    s = []  # 两两拆分


    for x in eval('itertools.product' + str(tuple(option))):
        cp.append(x)
        s.append([i for i in itertools.combinations(x, 2)])
    logger.info(f'笛卡尔积:{len(cp)},{cp}')
    logger.warning(f'对以上笛卡尔积的每个组合结果 进一步两两拆分:拆分出来的数量：{len(s)}\n{s}')

    del_row = []
    bar(0)
    s2 = copy.deepcopy(s)
    for i in range(len(s)):  # 对每行用例进行匹配
        if (i % 100) == 0 or i == len(s) - 1:
            bar(int(100 * i / (len(s) - 1)))
        t = 0
        for j in range(len(s[i])):  # 对每行用例的两两拆分进行判断，是否出现在其他行
            flag = False
            for i2 in [x for x in range(len(s2)) if s2[x] != s[i]]:  # 找同一列
                if s[i][j] == s2[i2][j]:
                    t = t + 1
                    flag = True
                    break
            if not flag:  # 同一列没找到，不用找剩余列了
                break
        if t == len(s[i]):
            del_row.append(i)
            s2.remove(s[i])

    res = [cp[i] for i in range(len(cp)) if i not in del_row]
    logger.info(f'过滤后:{len(res)}\n{res}')
    return res


def bar(i):
    """进度条"""
    c = int(i / 10)
    jd = '\r %2d%% [%s%s]'
    a = '■' * c
    b = '□' * (10 - c)
    msg = jd % (i, a, b)
    stdout.write(msg)
    stdout.flush()

if __name__ == '__main__':

    # 定义参数测试用例
    err_tem = {
        "正常情况": True,
        "字段不存在": False,
        "字段值为null": None,
        "字段值为数字0": 0,
        "字段值为数字": 123456,
        "字段值为字母": "asdfghh",
        "字段值为空字符串": "",
        "字段值为特殊字符": "!@#$%^&",
        "字段值超长_数字": 0000000000000000000000000,
        "字段值超长_字符串": "asdfdfjsjdkifjiosdjfoisdjofjsdofjosjfoghh"

    }

    # pl = [['M', 'O', 'P'], ['W', 'L', 'I'], ['C', 'E']]
    pl = [[['111111229980','465412'],['11111'],[' 0000000000000000000000000'],['']], ['New',''], ['xuefeng仓库',''], ['52005200','']]

    ret = parewise(pl)
    print(ret)

    for my_tuple in ret:
        key1, *other_keys = my_tuple
        # 瀚蓝，楼层，公司
        # keys = ['building', 'floor','company']
        keys = ['skuCodes', 'newOldSituations','warehouseIds','chargePerson']
        # skuCodes     newOldSituations        warehouseIds         chargePerson


        my_dict = {keys[i]: my_tuple[i] for i in range(len(my_tuple))}

        print(my_dict)


    # list_ = []
    # for i in ret:
    #     for key, value in i.items():
    #
    #
    #         a = i.values()
    #         list_.append(list(a)[0])
    #         # print(a)
    #

    # for i in a:
    #     print(i)



```

