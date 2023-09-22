# python基础：进程知多少

基础知识：

指定进程名，使用psutil 遍历系统所有进程id和名称，进而达到杀死进程的目的

```
# -*- coding: utf-8 -*-

# @Software: PyCharm
# @File: process.py
# @Author: xuefeng365
# @E-mail: 120158568@qq.com,
# @Site: www.51automate.cn
# @Time: 3月 29, 2023
# @Des: 指定进程名，使用psutil 遍历系统所有进程id和名称，进而达到杀死进程的目的
import os

import psutil


class Process_utils():

    def get_name_to_kill_process(self, name=''):
        '''
        :param name: 要杀死的进程名称
        :return:
        '''
        if name:
            # 获取所有进程id
            pids = psutil.pids()
            for pid in pids:
                p = psutil.Process(pid)
                # 获取进程名
                process_name = p.name()
                print(f'进程名称：{process_name}, pid:{pid}')
                if name in process_name:
                    self.kill_process(name=process_name,pid=str(pid))


    def kill_process(self, name='', pid=''):
        if name:
            #根据进程名杀死进程
            pro = f'taskkill /f /im {name}'
            os.system(pro)
            print(f'进程：{name} 已被杀死')

        if pid:
            #根据pid杀死进程
            process = f'taskkill /f /pid {pid}'
            os.system(process)
            print(f'进程id：{pid} 已被杀死')



a = Process_utils()
a.get_name_to_kill_process(name='Snipaste')


```



更多功能：比如通过psutil可以获取到所有进程的详细信息、内存信息、CPU信息、网络接口和链接信息，参考这里：https://www.liaoxuefeng.com/wiki/1016959663602400/1183565811281984

