2015-11-02 AndroidApk签名总结

# 使用Eclipse导出签名key文件
- 打开Eclipse，进行如下操作：
File-->Export-->Export Android Application-->选择对应的Project-->点击Next。
- 创建新的keystore文件
![这里写图片描述](http://img.blog.csdn.net/20151102142844683)
- 创建key
![这里写图片描述](http://img.blog.csdn.net/20151102143121202)
- 选择key作用到的apk文件
![这里写图片描述](http://img.blog.csdn.net/20151102143338008)
# 编写BAT签名脚本
编写脚本如下，请在jdk环境变量配置OK的前提下使用。
```
jarsigner -verbose -keystore test.key -storepass 111111 -signedjar demo_signed.apk demo.apk yiyixiaozhi
@pause
```
![这里写图片描述](http://img.blog.csdn.net/20151102144419619)
> 注：也可以修改签名后的apk文件名称，举例如下（apk名称末尾带上签名时候的系统时间）：
> ```
set/p option=请输入本目录待签名apk:
set str0="%date:~0,4%%date:~5,2%%date:~8,2%"
set str1="%time:~0,2%%time:~3,2%%time:~6,2%"
jarsigner -verbose -keystore test.key -storepass 111111 -signedjar "%option:~0,-4%"_signed_"%str0%"_"%str1%%option:~-4%" "%option%" yiyixiaozhi
@pause
```
输出apk文件名称如下：demo_signed_20151102_145726.apk
# jarsigner命令介绍
| Jarsigner 选项 | 描述 |
|---|:-:|
| -keystore.keystore | 包含你私钥的存储文件 |
| -verbose |显示输出动作。|
|-sigalg |签名算法，用 SHA1withRSA. |
| -digestalg| 消息摘要算法，用 SHA1. |
|-storepass |存储文件的密码。<br>主要为了安全起见，如果没提供，jarsigner会提示你输入。这个密码不会存储在你的shell历史记录中。 |
|-keypass |私钥的密码。<br>主要为了安全起见，如果没提供，jarsigner会提示你输入。这个密码不会存储在你的shell历史记录中。|