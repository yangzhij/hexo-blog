---
title: zabbix配置邮件报警
date: 2019-02-18 17:47:41
tags:
---
//zabbix服务端安装
```
yum -y install sendmail
yum install mailx –y
systemctl start mail
systemctl start sendmail.service
```
//测试
```
echo "zabbix testmail" |mail -s "zabbix" zhangsan@163.com
```

//测试正常后，开始配置zabbix界面
![https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail1.png](https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail1.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail2.png](https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail2.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail3.png](https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail3.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail4.png](https://raw.githubusercontent.com/yangzhij/images2/master/zbx-mail4.png)
```
故障通知：服务器:{HOSTNAME1} {EVENT.NAME}！！！

告警主机:  {HOSTNAME1}<br>
告警时间:  {EVENT.DATE}  {EVENT.TIME} <br>
告警信息:  {EVENT.NAME}<br>
告警项目:  {TRIGGER.KEY1}<br>
事件ID:  {EVENT.ID}
-------------------------------------------------------------------------------
故障{EVENT.NAME}已恢复正常

故障主机：{HOST.NAME}<br>
恢复时间：{EVENT.RECOVERY.DATE} {EVENT.RECOVERY.TIME} <br>
当前状态：{EVENT.STATUS}
事件ID:  {EVENT.ID}
```
