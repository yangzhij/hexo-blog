---
title: python脚本涉及知识点
date: 2019-01-16 10:26:27
tags:
---
```
# 创建一个文件并写入内容

ge = "E:\PycharmProjects\ge.conf"           #文件路径
file = "www.test.com"             #文件内容
f = open(ge, 'w')    
f.write(file)
f.close()
```


#判断是否存在目录，不存在则创建
```
root = input("输入站点目录(例如:bokeqipai):").strip()
wwwroot="E:\PycharmProjects\\"         #路径
if os.path.isdir(root):         #目录是否存在
    print("存在%s文件夹"%root)
else:
    print("文件夹%s不存在"%root)
    print("创建文件夹%s"%root)
    os.mkdir(root)         #创建目录
    print("%s 文件夹创建完成"%root)
```


#re.sub是个正则表达式方面的函数，用来实现通过正则表达式，实现比普通字符串的replace更加强大的替换功能；
```
import re
inputStr1 = "hello 111 world 111"
replacedStr1 = inputStr1.replace("111", "222")              #把字符串中的111换成222
print(replacedStr1)

inputStr = "hello 123 world 456"
replacedStr = re.sub("\d+", "222", inputStr)              #把字符串中的123和456都换成222
print(replacedStr)

```
