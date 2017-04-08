2015-12-3 常用adb命令示例
#Android4.4以上屏幕录像方法
一些问题提交屏幕录像信息，可以大大降低沟通成本。录像方法示例如下：
```
>adb shell
$ screenrecord --size 1280x760 /sdcard/test.mp4
```
> 注：如果要降低屏幕录像清晰度，减小文件大小，也可以使用640x360分辨率来录像。

# 过滤关键字log
```
adb shell logcat -v time | find /i "keyword"
```
#修改Android设备时间
adb shell date -s "19701216.120503"
---
#查看activity使用的so库路径
```java
ActivityInfo ai;
ai = getPackageManager().getActivityInfo(getIntent().getComponent(), PackageManager.GET_META_DATA);
ai.applicationInfo.nativeLibraryDir
```
---
#测试Android某个应用（testapp）内存使用举例
脚本如下（以包名中包含testapp为例）：
```bat
#! /bin/sh
for ((;;)) do
{
adb shell procrank | grep testapp| sed 's/K//g'
}
done
```
#查看mali内存
```
cat /sys/kernel/debug/mali/memory_usage
```
#根据进程号查看内存占用
```
dumpsys meminfo 1814
```
#测试内存free
```
procrank
```
#查看SurfaceFlinger
```
dumpsys SurfaceFlinger
```
#自动发送键值脚本示例
```
#!/system/bin/sh
while((1));do
input keyevent 23
sleep 5
input keyevent 23
sleep 5
input keyevent 4
sleep 5
input keyevent 4
done
```