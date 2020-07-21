---
layout: mypost
title: 利用Python给PDF添加目录
categories: [Python,pdf]
---

##### 安装`pymupdf`模块

```
pip install pymupdf
```

##### 编写`add.py`

``` python
# -*- coding: UTF-8 -*-
# add.py
# by devxzh@qq.com or onexzh@gmail.com
import fitz
import re
import sys


def print_help():
    print("""
          Command is as follow:
          >>> python add.py test.pdf list.txt num
          where  'test.pdf' is the target file.
                 'list.txt' is the catalog file to be added.
                 'num' is an integer containing +/- ,the default is 0.
          """)


argc = len(sys.argv)  # Number of input parameters

if argc < 3:
    print_help()
    exit()
else:
    pdfName = sys.argv[1]  # pdf file name
    txtName = sys.argv[2]  # txt file name
    # Offset the deviation between the PDF page number and the actual page number
    offset = 0 if argc == 3 else int(sys.argv[3])

if '.pdf' in pdfName:
    try:
        doc = fitz.open(pdfName)
    except:
        print("Please check the pdf file name and try again!")
        exit()
else:
    print("source file must be .pdf !")
    exit()

toc = []  # doc.getToC()  # get contents


def countdot(str):
    """
    没有点是一级标题，1个点是二级标题，2个点是三级标题
    """
    n = 1
    for i in str:
        if i == '.':
            n += 1
    return n


def get_page(str):
    """
    :param str:
    :return: int page
    """
    page = re.findall(r"[0-9]{1,3}$", str)  # list
    return int(page[0])


def get_title(str):
    """
    :param str:
    :return: str title
    """
    return re.sub(r"[0-9]{1,3}$", '', str)


if '.txt' in txtName:
    try:
        contentFile = open(txtName, encoding='UTF-8', errors='ignore')
    except:
        print("Please check the txt file name and try again!")
        exit()

    lines = contentFile.readlines()  # get all lines

    for line in lines:  # get every line
        # Skip blank lines and comment lines
        if line[0] == '#' or len(line.split()) == 0:
            continue
        part = line.split()  # Split line into three elements
        # if len(part) != 3:  # 缺少一项或多出一项(多出空格导致)
        #     print("Error: {0} lose some element".format(part))
        #     exit()
        print(part)
        if '章' in part[0]:
            level = 1
        else:
            level = 2
        page = get_page(part[1]) + int(offset)
        title = get_title(part[1])

        # print("{0},{1},{2}".format(level, part[1], page))
        toc.append([level, title, page])

    doc.setToC(toc)  # set contents
    newName = eval(repr(pdfName).replace("\\", "-"))  # Remove the '\' in the path
    partName = newName.split("-")
    newName = 'new-' + partName[-1]
    doc.save(newName)
    print("Added successfully!")

else:
    print("catalog file must be .txt !")
    exit()
```

##### 编写目录文件`list.txt`

```txt
# =========================================================================
# 本应用主要针对扫(dao)描(ban)的pdf文件
# 目录可在各大图书网站(如豆瓣，京东，淘宝)爬取，并稍做修改，后续可能添加OCR和自动处理模块
# =========================================================================
# 目录的每一行 包含三个元素 (以空格为间隔)
# 第一个字符串(程序中转换为数字) 表示目录级别，如1表示一级标题，2表示二级标题
# 第二个字符串 表示目录名 该版本仅支持中间没有空格的名称如 开 始 ，则程序可能出错
# 第三个字符串表示 页码 (始于1)
# =========================================================================

章	封面1
章	扉页2
章	内容简介3
章	版权页3
章	前言4
章	目录6
第1章　Python、Django和HTTP9
1.1　Python语言9
1.2　Django框架12
1.3　HTTP概述14
第2章　Django基本知识25
2.1　启动Django服务25
2.2　Hello_World程序29
2.3　获取参数31
2.4　HttpRequest对象与HttpResponse对象35
2.5　setting.py的配置37
2.6　session和cookie50
2.7　Django的MTV开发模式框架57
2.8　Django的模型与数据库的管理58
2.9　Django的视图管理69
2.10　Django的模板管理74
2.11　基于Python_Requests类数据驱动的HTTP接口测试83
第3章　电子商务网站的实现100
3.1　需求描述100
3.2　数据Model设计101
3.3　用户信息模块103
3.4　商品信息模块142
3.5　购物车模块157
3.6　送货地址模块175
3.7　订单模块189
3.8　电子支付模块208
3.9　建立自定义的错误页面208
第4章　构建安全的网站213
4.1　密码的加密213
4.2　防止CSRF攻击214
4.3　权限操作的漏洞220
4.4　防止XSS攻击226
4.5　防止SQL注入226
章 正文结束227
章 参考文献228
```

##### 使用方法

1. 将`.pdf`文件放到`.py`同一目录

2. 打开终端，切换到该目录

3. 使用如下指令

   ```powershell
   python add.py name.pdf list.txt num
   ```

   