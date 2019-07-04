---
title: python之用户认证
date: 2019-01-16 10:30:53
tags:
---
同目录下创建account.txt及account_lock.txt
允许用户输入三次，三次都错误，则锁定账户。

```
import sys,os
count = 0
name_list = []
while count < 3:
    name = input("请输入用户名：")
    lock_file = open('account_lock.txt', 'r+')
    lock_list = lock_file.readlines()

    for lock_line in lock_list:
        lock_line = lock_line.strip('\n')
        if name == lock_line:
            sys.exit('用户 %s 已经被锁定,请联系管理员解锁.' % name)
    user_file = open('account.txt','r')
    user_list = user_file.readlines()
    for user_line in user_list:
        (user,password) = user_line.strip('\n').split()
        name_list.append(user_line)
        #print("--------", name_list)                  #不想输出用户名密码可以注释此行
        if name == user:
            i = 0
            while i < 3:
                passwd = input('请输入您的密码：')
                if passwd == password:
                    print('欢迎 %s 登录' % name)
                    sys.exit(0)
                else:
                    if i < 2:
                        print('用户 %s 密码错误，请重新输入，还有 %d 次机会.' % (name,2 - i))
                i += 1
            else:
                lock_file.write(name + '\n')
                sys.exit('用户 %s 输错密码三次，用户将被锁定并退出，请联系管理员解锁.' % name)
        else:
            pass
    else:
        if count < 2:
            print('用户 %s 不存在，请重新输入，还有 %d 次机会' % (name,2 - count))
    count += 1
else:
    lock_file.write(name + '\n')
    sys.exit('用户 %s 可能是个山炮，退出应用' % name)

lock_file.close()
user_file.close()


```