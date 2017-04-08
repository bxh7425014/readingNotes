2015-10-30 Windows批处理编写心得.md
[TOC]
# bat 批处理－取年、月、日、时、分、秒、毫秒
取年份：echo %date:~0,4%
取月份：echo %date:~5,2%
取日期：echo %date:~8,2%
取星期：echo %date:~10,6%
取小时：echo %time:~0,2%
取分钟：echo %time:~3,2%
取秒：echo %time:~6,2%
取毫秒：echo %time:~9,2%
#​Windows下直接定位到文件/文件夹bat批处理编写
- 打开资源管理器
```
explorer /e,C:\Documents and Settings\Administrator\桌面\Inbox
```
- 打开文件夹
```
explorer ,C:\Documents and Settings\Administrator\桌面\Inbox
```
- 直接定位文件夹
```
explorer /select,C:\Documents and Settings\Administrator\桌面\Inbox
```
- 直接定位文件
```
explorer /select,C:\Documents and Settings\Administrator\桌面\Inbox\字母排序新.jpg
```
# 使用adb自动获取log
```
del log.log
adb logcat -c
adb logcat -v time log.log > log.log
@pause
```
> logcat -v time 打印Android系统当前时间到log中
# adb发送广播Broadcast
```
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED
@pause
```
>可以更精确的发送到某个package，如下：
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED -c android.intent.category.HOME -n package_name/class_name
# 编写自动延时发送循环键值的bat批处理
```java
@echo off
:start
set /a var+=1
echo %var%
  echo inputKeyEvent:Left(21)
  adb shell input keyevent 21
  start /min /w mshta vbscript:setTimeout("window.close()",5000)
  adb shell procrank >> .\procrank.log
  echo inputKeyEvent:Right(22)
  adb shell input keyevent 22
  start /min /w mshta vbscript:setTimeout("window.close()",5000)
  adb shell procrank >> .\procrank.log
if %var% leq 100000 GOTO start
pause
```