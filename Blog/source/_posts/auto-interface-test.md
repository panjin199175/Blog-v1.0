---
title: 接口自动化测试
date: 2020-01-13 21:25:33
tags:
  - Python
categories:
  - 接口测试
---

# **自动化测试模型**
自动化测试模型可以看作是自动化测试框架设计的思想，这里介绍4种模型：线性测试、模块化驱动测试、数据驱动测试、关键字驱动测试。
1.线性测试：每个测试脚本都是完整且独立的，且不产生依赖与调用，缺点显而易见，开发和维护成本很高
2.模块化驱动测试：借鉴了编程语言中模块的思想（比如Python的包），把重复的操作当作了独立的公共模块，当用例执行过程中需要用到这一模块操作时则被调用，这样就最大程度地消除了重复，从而提高了开发效率和可维护性。
3.数据驱动测试：顾名思义就是数据的改变从而驱动自动化测试的执行，就是数据的参数化，因为输入的数据不同从而引起输出结果的不同，实现数据与脚本的分离，进一步增强了脚本的复用性。
4.关键字驱动测试：就是在数据驱动的基础上，把“数据”换成“关键字”，通过关键字来引起测试结果的改变。
# **自动化接口框架设计**
选用模型：数据驱动+关键字驱动+模块化驱动
设计理由：
1.使用数据驱动，实现数据与脚本分离的，增强脚本复用性
   2.使用关键字驱动，通过关键字来查找数据，可以使测试所需数据从单一文本转移，由多个文件存储数据，看起来不是很笨重
   3.使用模块化驱动，对数据的重复操作的功能可以写成公共模块，举例：对excel的操作数据模块，构造请求模块，发送测试报告邮件模块。
设计方案：
数据驱动+关键字驱动
包名：case_data
模块名称:interface.xlsx,case.json
取名为interface.xlsx的excel文件来存储测试用例,case.json文本存储关键字的值
字段设计：
1.case_id ：记录用例号
2.名称：case_id说明
3.url：请求地址
4.run：有两个值，yes 和 no,判断用例是否需要执行
5.请求方式：有GET 和 POST两个值，来判断使用哪一种请求方式
6.headers：由关键字驱动，其值存储在case.json文本中，headers为空，则表示不需要headers
7.cookie：有yes 和 no两个值，来判断是否需要携带cookie
8.依赖case：某些请求的headers或body中可能需要上一个请求的响应中的值，依赖case指出需要依赖哪一个case
9.返回依赖数据：指被依赖的case的字段，一般json格式的响应结果为嵌套式的dictionary，这里举例关键字：info.toke，
假设
```python
respose={'data'：'mini',
	    infor:'{
                'token':'unknown',
                 'num':'3'
          }
```
我们要取的值为response['info']['token']，由方法来实现
10.数据依赖字段：若该字段为：Authorization则’Authorization':'response['infor']['token']'，根据8，9，获取上一个请求响应中所需数据
11.依赖需要：此处取3个值，headers、data、空
headers：表示数据依赖放在headers的字典里
data，表示数据依赖放在body的字典里
空：表示不需要依赖数据
12 请求参数：body中的数据，由关键字驱动
13 预期结果：与实际结果相比较
14 实际结果：有3个值 pass 、fail、空，脚本运行前或run=no的用例该项为空，其值由响应结果和预期结果断言后写入excel所得
模块化驱动：
包名：tools
模块名称：operating_excel.py
前提：
1.学习python操作excel：
https://www.cnblogs.com/zhoujie/p/python18.html
我们只要学会：
1.获取sheet的行数 sheet.nrows：即得到case个数，要减一，第一行不算
2.获取单元格的内容 sheet.cell_value(row,col)，获得参数的值

 2.学习python写入已存在数据的eccel：http://blog.csdn.net/hqzxsc2006/article/details/51768351
为什么是已存在：不然每次写入，都会把之前的内容清空。
模块名称：operating_json.py
由关键字驱动的真实值存储在json中，我们通过引入operating_excel模块
可以得到获取关键字的方法，然后通过oprating_json，可以由关键字得到真实的参数
前提：
学习对json的读操作：https://www.cnblogs.com/eejron/p/4708980.html
我们只要学会读取json文件，读取后取值，json用法跟dictionary类似

模块名称：sendmail.py
用例执行完毕后，可以向指定收件群体发送邮件，汇报接口测试成功率
可以设置成由接口错误来触发
前提：
学习使用smtplib发送邮件：https://www.cnblogs.com/houzhizhe/p/7458960.html
我们只要学会发送普通邮件就可以了
包名：request
模块名称：build_request.py
通过引入build_request模块获取构造请求的方法

包名：data
模块名称：data_set.py
设计通用常量，定义sheet中每个字段中的所在列的值

模块名称：data_get.py
通过引入data_get，我们可以得到sheet中的每个单元格的值
我们已经定义了operating_excel和operating_json的方法，从data_set中也知道了每个字段的所在列，我们就可以通过遍历每一行（row，col确定一个单元格内容），由operating_excel获取每个单元格的值，由operating_json通过关键字获取json文件中的值(operating_excel、operating_json、data_set是为data_get做辅助的）

模块名称：dependent_case.py
设计：根据依赖case_id，读取该case_id所在行，将它的所有数据再读取一遍，构造请求，获取响应数据，根据依赖需要，最终获得依赖数据
包名：main
模块名称：run_test.py
真正执行的测试用例的模块，实现逻辑
[![逻辑图](https://i.loli.net/2019/09/25/Fn5IJerquhLtafo.jpg "逻辑图")](https://i.loli.net/2019/09/25/Fn5IJerquhLtafo.jpg "逻辑图")
# Module Tree
[![项目工程树](https://i.loli.net/2019/09/25/ZUbNESMtHy8rqXe.png "项目工程树")](https://i.loli.net/2019/09/25/ZUbNESMtHy8rqXe.png "项目工程树")


##### 谢谢观赏
