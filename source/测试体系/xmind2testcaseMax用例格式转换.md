# xmind2testcaseMax用例格式转换

## xmind2testcaseMax工具代码

更新日志：20250521

1. 基于xmind2testcase改造，新增对最新版本（[25.01.01061](https://dl3.xmind.cn/Xmind-for-Windows-x64bit-25.01.01061-202501070746.exe)）的支持；
2. 一个步骤可支持多个预期结果，并可现实每个预期结果执行结果（失败、阻塞）；
3. 新增重要功能：将用例概要 转换为 前置条件，xmind看起来更直观；
4. 新增对excle的支持；
5. 自定义导出模板以适应公司meegle/jira导入；
6. 优化web和cli代码，点点页面即可导出不同需求的用例文件；
7. 文件导出内容的其他小优化；

---

## 1. 工具准备

### 1.1 安装 python

pass

### 1.2 安装 xmind2testcase

pass

### 1.3 安装 第三方包： openpyxl
pass

## 2.自定义导出模版+用例步骤支持多个预期结果

### 2.1 zentao.py 文件修改

python安装路径的**\Lib\site-packages\xmind2testcase**文件夹下找到**zentao.py**文件并打开。



**自定义导出字段**、新增导出excle文件、修改优先级、处理导出文件有空行问题、去除用例标题中的空格、处理用例步骤预期结果序号后多一个空格等自行参考 zentao.py 文件即可。

```
def xmind_to_zentao_csv_file(xmind_file, label='meegle', type='csv'):
    """Convert XMind file to a zentao csv file"""
    xmind_file = get_absolute_path(xmind_file)
    logging.info('Start converting XMind file(%s) to zentao file...', xmind_file)
    testcases = get_xmind_testcase_list(xmind_file)

    if label == 'meegle':
        fileheader = ["用例名称", "模块", "Priority", "用例类型", "前置条件", "用例步骤", "预期结果"]
    elif label == 'report':
        fileheader = ["编号", "功能模块", "用例标题", "优先级", "前置条件", "步骤", "预期结果", "创建者", "编写人更新时间", "执行状态", "执行状态"]
    else:
        pass
        # fileheader = ["用例标题", "模块", "优先级", "用例类型", "前置条件", "步骤", "预期结果"]

    zentao_testcase_rows = [fileheader]
    for testcase in testcases:
        # print(f'testcase:{testcase}')
        row = gen_a_testcase_row(testcase, label=label)
        zentao_testcase_rows.append(row)

    # Generate CSV file
    zentao_csv_file = xmind_file[:-6] + '.csv'
    if os.path.exists(zentao_csv_file):
        os.remove(zentao_csv_file)

    with open(zentao_csv_file, 'w', newline='', encoding='utf8') as f:
        writer = csv.writer(f)
        writer.writerows(zentao_testcase_rows)
        logging.info('Convert XMind file(%s) to a zentao csv file(%s) successfully!', xmind_file, zentao_csv_file)

    # Generate Excel file
    zentao_excel_file = xmind_file[:-6] + '.xlsx'
    if os.path.exists(zentao_excel_file):
        os.remove(zentao_excel_file)

    wb = Workbook()
    ws = wb.active
    ws.title = "Test Cases"

    for row in zentao_testcase_rows:
        ws.append(row)

    wb.save(zentao_excel_file)
    logging.info('Convert XMind file(%s) to a zentao excel file(%s) successfully!', xmind_file, zentao_excel_file)

    if type == 'csv':
        return zentao_csv_file

    elif type == 'excel':
        return zentao_excel_file
    else:
        raise ValueError("Unsupported file type. Please choose 'csv' or 'excel'.")

```

特别指出，以上方法，通过` label`自由控制导出模板文件的标题；通过`type`控制导出文件类型（默认csv，可选excle），后面我们会改造web端`type`的值 以支持不同类型文件导出，效率更高。



### 2.3 parser.py 文件的修改

python安装路径的**\Lib\site-packages\xmind2testcase**文件夹下找到**parser.py**文件并打开，修改以下方法

##### 2.3.1 支持：一个步骤多个预期结果

1. 新增一个步骤支持多个预期结果（原有工具一个步骤只能一个预期结果）；

2. 步骤没有预期，默认填充'/'

```
def parse_a_test_step(step_dict):
    test_step = TestStep()
    test_step.actions = step_dict['title']

    expected_topics = step_dict.get('topics', [])
    if expected_topics:  # 有预期结果
        # 支持多个预期结果
        for expected_topic in expected_topics:
            # expected_topic = expected_topics[0]
            # test_step.expectedresults = expected_topic['title']  # 一个测试步骤操作，一个测试预期结果
            markers = expected_topic['markers']
            result = get_test_result(markers)
            # test_step.result = get_test_result(markers)

            a = f"{expected_topic['title']}; "
            if result:
                a = f"{expected_topic['title']} - 【{result}】; "
            test_step.expectedresults += a
            # print(f'预期结果:{test_step.expectedresults}')
    else:  # 只有步骤无预期结果
        # 默认填充'/'
        test_step.expectedresults = '/'
        markers = step_dict['markers']
        test_step.result = get_test_result(markers)

    logging.debug('finds a teststep: %s', test_step.to_dict())
    return test_step
```

##### 2.3.2 预期结果改造 （xmind执行用例可显示 每条预期执行情况：阻塞或失败）

```
def get_test_result(markers):
    """test result: non-execution:0, pass:1, failed:2, blocked:3, skipped:4"""
    if isinstance(markers, list):                                       # 完成符号                 喜欢
        if 'symbol-right' in markers or 'c_simbol-right' in markers or 'task-done' in markers or 'c_symbol_like' in markers:    # 通过
            result = '通过'                                              # 不喜欢
        elif 'symbol-wrong' in markers or 'c_simbol-wrong' in markers or 'c_symbol_dislike' in markers:  # 失败
            result = '失败'                                              # 红色感叹号
        elif 'symbol-pause' in markers or 'c_simbol-pause' in markers or 'symbol-exclam' in markers:  # 阻塞
            result = '阻塞'
        # elif 'symbol-minus' in markers or 'c_simbol-minus' in markers:  # 跳过
        #     result = '跳过'
        else:
            result = ''
    else:
        result = ''

    return result
    
```

![image.png](http://biji.51automate.cn/blogs/img/202503251416280.png)


##### 2.3.1 去除用例标题中的空格

```python
def gen_testcase_title(topics):
    """Link all topic's title as testcase title"""
    titles = [topic['title'] for topic in topics]
    titles = filter_empty_or_ignore_element(titles)
    # when separator is not blank, will add space around separator, e.g. '/' will be changed to ' / '
    #下面模块和功能是用"空格"分割，如用"/"分隔，则替换为：separator = '/'
    #separator = '/'
    separator = config['sep']
    if separator != ' ':
        # 修改前
        #separator = ' {} '.format(separator)
        # 修改后
        separator = f'{separator}'
    return separator.join(titles)

```

效果图![image-20250124143410234](http://biji.51automate.cn/blogs/img/202501241434024.png)

### 2.4 支持新版xmind

1. 安装`xmindparser`库（目前我的版本是：v1.0.9）

2. python安装路径的**\Lib\site-packages\xmind2testcase**文件夹下找到**utils.py**文件并打开，修改以下方法

   ```
   from xmindparser import xmind_to_dict,is_xmind_zen
   
   def get_xmind_testsuites(xmind_file):
       """Load the XMind file and parse to `xmind2testcase.metadata.TestSuite` list"""
       xmind_file = get_absolute_path(xmind_file)
       # 新增 判断是否为新版xmind(zen及其之后的版本)
       if is_xmind_zen(xmind_file):
           print("yes,新版xmind")
           xmind_content_dict = xmind_to_dict(xmind_file)
       else:
           workbook = xmind.load(xmind_file)
           xmind_content_dict = workbook.getData()
   ```

3. python安装路径的**\Lib\site-packages\xmindparser**文件夹下找到**zenreader.py**文件并打开，新增以下方法

   ```
   def update_notes_with_summary(topics):
       """
       递归遍历主题结构，将 summary 内容合并到 notes.plain.content 中
       """
       if isinstance(topics, list):
           for topic in topics:
               update_notes_with_summary(topic)
       elif isinstance(topics, dict):
           # 处理当前主题的 summary
           if 'summary' in topics:
               summary_content = "\n".join([summary['title'] for summary in topics['summary']])
               if 'notes' in topics:
                   if 'plain' in topics['notes']:
                       topics['notes']['plain']['content'] += "\n" + summary_content
                   else:
                       topics['notes']['plain'] = {'content': summary_content}
               else:
                   topics['notes'] = {'plain': {'content': summary_content}}
   
           # 处理 summaries
           if 'summaries' in topics and 'children' in topics and 'attached' in topics['children']:
               for summary_info in topics['summaries']:
                   range_str = summary_info['range']
                   topic_id = summary_info['topicId']
   
                   # 解析 range_str 为范围
                   if ',' in range_str:
                       start, end = map(int, range_str.strip('()').split(','))
                   else:
                       start = int(range_str.strip('()'))
                       end = start + 1
   
                   # 找到对应的 summary
                   summary_title = None
                   if 'summary' in topics['children']:
                       for summary in topics['children']['summary']:
                           if summary['id'] == topic_id:
                               summary_title = summary['title']
                               break
   
                   if summary_title:
                       for i in range(start, end+1):
                           if i < len(topics['children']['attached']):
                               attached_topic = topics['children']['attached'][i]
                               if 'notes' in attached_topic:
                                   if 'plain' in attached_topic['notes']:
                                       attached_topic['notes']['plain']['content'] += "\n" + summary_title
                                   else:
                                       attached_topic['notes']['plain'] = {'content': summary_title}
                               else:
                                   attached_topic['notes'] = {'plain': {'content': summary_title}}
   
           # 递归处理子主题
           if 'children' in topics:
               if 'attached' in topics['children']:
                   for child in topics['children']['attached']:
                       update_notes_with_summary(child)
           # 特别处理 rootTopic
           if 'rootTopic' in topics:
               update_notes_with_summary(topics['rootTopic'])
   ```

4. 修改`sheet_to_dict`方法

   ```
   def sheet_to_dict(sheet):
       """convert a sheet to dict type."""
       # ---------- 新增开始  ---------
       print(f"sheet:{sheet}")
       if sheet:
           # 将sheet测试用例的概要信息变为前置条件
           update_notes_with_summary(sheet)
       # ---------- 新增结束  ---------
   
       topic = sheet['rootTopic']
       result = {'title': sheet['title'], 'topic': node_to_dict(topic), 'structure': get_sheet_structure(sheet)}
   ```





### CLI命令配置

![image-20250124154524401](http://biji.51automate.cn/blogs/img/202501241545660.png)

![image-20250124154638703](http://biji.51automate.cn/blogs/img/202501241546054.png)

```
        elif len(sys.argv) == 3 and sys.argv[2] == '-excel2meegle':
            kpay_excel2meegle_file = xmind_to_zentao_csv_file(xmind_file, label='meegle', type='excel')
            logging.info('Convert XMind file to excel2meegle excel file successfully: %s', kpay_excel2meegle_file)
        elif len(sys.argv) == 3 and sys.argv[2] == '-excel2report':
            kpay_excel2report_file = xmind_to_zentao_csv_file(xmind_file, label='report', type='excel')
            logging.info('Convert XMind file to kpay excel2report file successfully: %s', kpay_excel2report_file)
```

 转化命令

```
xmind2testcase /path/to/testcase.xmind -excel2meegle

xmind2testcase /path/to/testcase.xmind -excel2report

xmind2testcase /path/to/testcase.xmind -json
```



### Web配置

效果![image-20250124154933231](http://biji.51automate.cn/blogs/img/202501241549916.png)

![image-20250124155054129](http://biji.51automate.cn/blogs/img/202501241550498.png)



1. web前端应用：新增路由

2. python安装路径的**\Lib\site-packages\webtool**文件夹下找到**application.py**文件并打开，新增以下方法

   ```
   @app.route('/<filename>/to/meegle_excel')
   def download_kpay_xlsx_file_by_pandas_to_meegle(filename):
       full_path = join(app.config['UPLOAD_FOLDER'], filename)
       if not exists(full_path):
           abort(404)
       kpay_xlsx_file = xmind_to_zentao_csv_file(full_path, label='meegle', type='excel')
   
       filename = os.path.basename(kpay_xlsx_file) if kpay_xlsx_file else abort(404)
       return send_from_directory(app.config['UPLOAD_FOLDER'], filename, as_attachment=True)
   
   
   @app.route('/<filename>/to/report_excel')
   def download_kpay_xlsx_file_by_pandas_to_report(filename):
       full_path = join(app.config['UPLOAD_FOLDER'], filename)
       if not exists(full_path):
           abort(404)
       kpay_xlsx_file = xmind_to_zentao_csv_file(full_path, label='report', type='excel')
   
       filename = os.path.basename(kpay_xlsx_file) if kpay_xlsx_file else abort(404)
       return send_from_directory(app.config['UPLOAD_FOLDER'], filename, as_attachment=True)
   
   ```

   ![image-20250124155730075](http://biji.51automate.cn/blogs/img/202501241557689.png)

1. 导出文件-首页入口标签
   Python 安装路径的**\Lib\site-packages\webtool\templates**文件夹下找到**index. Html**文件并修改
   ![image-20250124160306489](http://biji.51automate.cn/blogs/img/202501241603426.png)

   ```
   <td><a href="{{ url_for('uploaded_file',filename=record[1]) }}">XMIND</a> |
                                   <a href="{{ url_for('download_zentao_file',filename=record[1]) }}">CSV</a> |
                                   <a href="{{ url_for('download_kpay_xlsx_file_by_pandas_to_meegle',filename=record[1]) }}">Excel-meegle</a> |
                                   <a href="{{ url_for('download_kpay_xlsx_file_by_pandas_to_report',filename=record[1]) }}">Excel-report</a> |
                                   <a href="{{ url_for('download_testlink_file',filename=record[1]) }}">XML</a> |
                                   <a href="{{ url_for('preview_file',filename=record[1]) }}">PREVIEW</a> |
                                   <a href="{{ url_for('delete_file',filename=record[1], record_id=record[4]) }}">DELETE</a>
                               </td>
   ```

   

2. 导出文件-预览页入口标签
   Python 安装路径的**\Lib\site-packages\webtool\templates**文件夹下找到**preview. Html**文件并修改
   ![image-20250124160148261](http://biji.51automate.cn/blogs/img/202501241601711.png)

   ```
       <h2>
           TestSuites: {{ suite_count }} / TestCases: {{ suite | length }}
           / <a href="{{ url_for('download_zentao_file', filename=name) }}">Get Zentao CSV</a>
           / <a href="{{ url_for('download_kpay_xlsx_file_by_pandas_to_meegle',filename= name) }}">Get KPay Excel-meegle</a>
           / <a href="{{ url_for('download_kpay_xlsx_file_by_pandas_to_report',filename= name) }}">Get KPay Excel-report</a>
           / <a href="{{ url_for('download_testlink_file', filename=name) }}">Get TestLink XML</a>
           / <a href="{{ url_for('index') }}">Go Back</a>
       </h2>
   ```


### 完整版二开文件
下载链接：
https://xuefeng365.lanzouo.com/b00l1k6rpc
密码:axiv

