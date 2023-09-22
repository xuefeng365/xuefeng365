# python 时间戳、时间字符串转换、时区处理方法

## 个人封装

```python
# -*- coding: utf-8 -*-

# @Software: PyCharm
# @File: log.py
# @Author: xuefeng365
# @E-mail: 120158568@qq.com,
# @Site: www.51automate.cn
# @Time: 12月 28, 2022

from loguru import logger

import time
import datetime

class Operate_time():

    def now_time(self,str_ = '%Y-%m-%d %H:%M:%S'):
        '''
        :return: 当前时间字符串
        '''
        return datetime.datetime.now().strftime(str_)

    def time_str_to_second_timestamp(self,time_str, format_str='%Y-%m-%d %H:%M:%S'):
        '''
        时间字符串>毫秒级时间戳
        @param time_str: 时间字符串
        @param format_str: 时间格式化字符串
        @return: float类型，时间戳
        '''
        # 转为秒级时间戳
        t3_seconds = time.mktime(time.strptime(time_str, format_str))
        return t3_seconds

    def time_str_to_millisecond_timestamp(self, time_str, format_str='%Y-%m-%d %H:%M:%S'):
        '''
        时间字符串>级时间戳
        @param time_str: 时间字符串
        @param format_str: 时间格式化字符串
        @return: float类型，时间戳
        '''
        t3_seconds = self.time_str_to_second_timestamp(time_str, format_str=format_str)
        # 转为毫秒级
        t3_milliseconds = float(str(t3_seconds * 1000).split(".")[0])
        return t3_milliseconds

    def timestamp_to_time_str(self,timestamp:float,type='seconds',format_str='%Y-%m-%d %H:%M:%S'):
        '''
        @return:
        @param timestamp: float，时间戳
        @param type: 默认转化秒级时间戳。 type=milliseconds，可转换毫秒级时间戳
        @param format_str: 格式化时间字符戳
        @return: 时间字符串
        '''
        # t = '%d-%02d-%02d %02d:%02d:%02d' % timestamp
        # # t = '%d-%02d-%02d %02d:%02d:%02d' % time.localtime()[:6]
        # # 获得当前时间时间戳
        # now = int(time.time())

        if type and type in 'seconds':
            print(111)
            now = float(timestamp)
        elif type and type in 'milliseconds':
            print(222)

            now = float(timestamp/1000)
        else:
            raise '请输入正确的 时间戳类型，有2种（seconds、milliseconds）'
        print(now)

        timeArray = time.localtime(now)
        time_str = time.strftime(format_str, timeArray)
        return time_str

    def time_to_timestamp(self, time_zone='', interval:int=0, type='seconds', format_str='%Y-%m-%d %H:%M:%S'):
        '''
        @param interval: int 间隔
        @param type: weeks、days、hours、minutes、seconds、milliseconds、microseconds
        @param time_zone: 选择时区  默认当前时区， 或者 0时区
        @param format_str:
        @return: list  [秒级时间戳，毫秒级时间戳]
        '''

        if time_zone=='0':
            # 0时区 时间
            t = datetime.datetime.utcnow()
        else:
            #  当前时区时间
            t = datetime.datetime.now()

        if type=='weeks':
            t3 = (t + datetime.timedelta(weeks=interval)).strftime(format_str)
        elif type=='days':
            t3 = (t + datetime.timedelta(days=interval)).strftime(format_str)
        elif type=='hours':
            t3 = (t + datetime.timedelta(hours=interval)).strftime(format_str)
        elif type=='minutes':
            t3 = (t + datetime.timedelta(minutes=interval)).strftime(format_str)
        elif type=='seconds':
            t3 = (t + datetime.timedelta(seconds=interval)).strftime(format_str)
        elif type=='milliseconds':
            t3 = (t + datetime.timedelta(milliseconds=interval)).strftime(format_str)
        elif type=='microseconds':
            t3 = (t + datetime.timedelta(microseconds=interval)).strftime(format_str)
        else:
            t3 = (t + datetime.timedelta(seconds=interval)).strftime(format_str)


        # 转为秒级时间戳
        t3_seconds = time.mktime(time.strptime(t3, format_str))
        # 转为毫秒级
        t3_milliseconds = float(str(t3_seconds * 1000).split(".")[0])
        return [t3_seconds, t3_milliseconds]

    def time_to_time_str(self, time_zone='', interval:int=0, type='seconds', format_str='%Y-%m-%d %H:%M:%S'):
        '''
        @param interval: int 间隔
        @param type: weeks、days、hours、minutes、seconds、milliseconds、microseconds
        @param time_zone: 选择时区  默认当前时区， 或者 0时区
        @param format_str:
        @return: list  [秒级时间戳，毫秒级时间戳]
        '''

        if time_zone=='0':
            # 0时区 时间
            t = datetime.datetime.utcnow()
        else:
            #  当前时区时间
            t = datetime.datetime.now()

        if type=='weeks':
            t3 = (t + datetime.timedelta(weeks=interval)).strftime(format_str)
        elif type=='days':
            t3 = (t + datetime.timedelta(days=interval)).strftime(format_str)
        elif type=='hours':
            t3 = (t + datetime.timedelta(hours=interval)).strftime(format_str)
        elif type=='minutes':
            t3 = (t + datetime.timedelta(minutes=interval)).strftime(format_str)
        elif type=='seconds':
            t3 = (t + datetime.timedelta(seconds=interval)).strftime(format_str)
        elif type=='milliseconds':
            t3 = (t + datetime.timedelta(milliseconds=interval)).strftime(format_str)
        elif type=='microseconds':
            t3 = (t + datetime.timedelta(microseconds=interval)).strftime(format_str)
        else:
            t3 = (t + datetime.timedelta(seconds=interval)).strftime(format_str)
        return t3


a = Operate_time()
# 时间转时间戳 list [秒级，毫秒级] (可指定获取0时区或当前时区、可快速获取距离当前时间一定间隔的时间戳)
logger.info(a.time_to_timestamp())

# 时间转时间字符串(可指定获取0时区或当前时区、可自定义时间字符串格式、可快速获取距离当前时间一定间隔的时间字符串)
logger.info(a.time_to_time_str(time_zone='', interval=0, format_str='%Y-%m-%d %H:%M:%S'))

# 获取当前时间
logger.info(a.now_time())

# 时间字符串 转 秒级时间戳
logger.info(a.time_str_to_second_timestamp('2022-12-28 20:25:43'))
# 时间字符串 转 毫秒级时间戳
logger.info(a.time_str_to_millisecond_timestamp('2022-12-28 20:25:43'))

```

## Faker库

利用 Faker生成常用的测试数据

官方文档：https://faker.readthedocs.io/en/stable/

```python
from faker import Faker


fake = Faker(locale="zh_CN")

print('姓名：', fake.name())
print('手机号：', fake.phone_number())

phones = [fake.phone_number() for _ in range(5)]  # 列表推导，把生成的数据组成一个列表
print('批量生成手机号：', phones)

print('邮箱：', fake.email())
print('MD5：', fake.md5())
print(fake.items())

print('年月日：', fake.date(pattern=' %Y-%m-%d'))

print('随机年份：', fake.year())

print('随机月份：', fake.month())

print('随机几号：', fake.day_of_month())

print('随机星期数：', fake.day_of_week())

print('随机时间字符串1：', fake.time(pattern='%Y-%m-%d %H:%M:%S'))
print('随机时间字符串2：', fake.time(pattern='%H:%M:%S'))

# -30y是过去30年前为开始日期，end_date表示结束到今天

print('过去某一天：', fake.date_between(start_date="-30y", end_date="today"))

print('今天：', fake.date_between_dates())  # 今天


print('日期和时间：', fake.date_time())  # 2021-05-14 19:36:00

print('当前日期时间：', fake.date_time_between_dates())

# print('某个区间内随机日期时 间：', fake.date_time_between_dates(datetime_start=datetime(1999, 2, 2, 10, 30, 20), dat
# etime_end = datetime(2000, 2, 2, 10, 30, 20)))

print('未来30天内的随机日期：', fake.future_date(end_date="+30d"), f'类型是：{type(fake.future_date(end_date="+30d"))},', str(fake.future_date(end_date="+30d")),'已转成字符串')

print('未来30天内的随机日期时间：', fake.future_datetime(end_date="+30d"),
      f'类型是：{type(fake.future_datetime(end_date="+30d"))}')  # 未来日期和时间)

print('过去的日期：', fake.past_date(start_date="-30m"))  # 过去日期

print('过去的日期时间：', fake.past_datetime(start_date="-30d"))  # 过去日期和时间

print('时间戳：', fake.unix_time(), f'类型是：{type(fake.unix_time())}')
print('时间戳(时分秒)：', fake.time(), f'类型是：{type(fake.time())}')
print('随机md5：', fake.md5(), f'类型是：{type(fake.md5())}')


```

