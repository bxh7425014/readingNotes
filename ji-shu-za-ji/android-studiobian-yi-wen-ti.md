2016-8-9 解决Android-Studio编译报错问题，找不到org-apache-http-legacy

在使用Android Studio编译app时，系统提示：```
Warning: Unable to find optional library: org.apache.http.legacy
```
注：Android Sdk 6.0
这个其实是因为缺少一个json配置文件导致的问题。文件位置：```
...AndroidSDK\platforms\android-23\optional
```
在此位置添加一个文件optional.json，内容如下：
```
[
  {
    "name": "org.apache.http.legacy",
    "jar": "org.apache.http.legacy.jar",
    "manifest": false
  }
]

```

参考网址：[Unable to find optional library - org.apache.http.legacy](http://www.mysysadmintips.com/other/programming/673-unable-to-find-optional-library-org-apache-http-legacy)