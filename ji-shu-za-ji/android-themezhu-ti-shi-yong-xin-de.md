2015-10-30 Android Theme主题使用心得.md
[TOC]
#Android设置透明主题
##透明Activity主题：
```xml
android:theme="@style/Theme.Translucent"
<!-- A theme that has a translucent background. Here we explicitly specify
that this theme is to inherit from the system's translucent theme,
which sets up various attributes correctly. -->
<style name="Theme.Translucent" parent="android:style/Theme.Translucent">
<item name="android:windowBackground">@drawable/translucent_background</item>
<item name="android:windowNoTitle">true</item>
<item name="android:colorForeground">#fff</item>
</style>
```
##3D界面（GLSurfaceView层）并把系统背景透过来显示：
```xml
android:theme="@style/Theme.CustomBackground_wallpager"
<style name="Theme.CustomBackground_wallpager" parent="@android:style/Theme.Wallpaper">
<item name="android:windowNoTitle">true</item>
<!-- 没标题 -->
<item name="android:windowFullscreen">true</item>
<!-- 全屏显示 -->
</style>
```