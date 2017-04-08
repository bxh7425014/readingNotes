2015-10-30 Android手机屏幕相关知识点.md

# 获取手机屏幕分辨率信息
```java
public static float PIXEL_DENSITY = 0.0f;
public static int PIXEL_HEIGHT;
public static int PIXEL_WIDTH;
DisplayMetrics metrics = new DisplayMetrics();
getWindowManager().getDefaultDisplay().getMetrics(metrics);
PIXEL_DENSITY = metrics.density;
PIXEL_HEIGHT = metrics.heightPixels;
PIXEL_WIDTH = metrics.widthPixels ;
```
# Android Drawable切图制作标准
## 屏幕密度级别
屏幕密度 dpi： 每英寸像素数
low:medium:high:extra-high:extra-extra-high=3:4:6:8:12
ldpi是120dpi，mdpi是160dpi，hdpi是240dpi，xhdpi是320dpi，xxhdpi是480dpi
## 手机常见分辨率
<table>
<tr><td>4:03</td><td>5:03</td><td>16:09</td></tr>
<tr><td>VGA 640*480 (Video Graphics Array)</td><td>WVGA 800*480 (Wide VGA)</td><td>FWVGA 854*480 (Full Wide VGA)</td></tr>
<tr><td>QVGA 320*240 (Quarter VGA)</td><td></td><td>HD 1920*1080 High Definition</td></tr>
<tr><td>HVGA 480*320 (Half-size VGA)</td><td></td><td>QHD 960*540</td></tr>
<tr><td>SVGA 800*600 (Super VGA)</td><td></td><td>720p 1280*720 标清</td></tr>
<tr><td></td><td></td><td>1080p 1920*1080 高清</td></tr>
<tr><td></td></tr>
</table>
## 分辨率对应DPI(制作切图请参考下面的后5项制作)
<table>
<tr><td>ldpi</td><td>mdpi</td><td>hdpi</td><td>xhdpi</td><td>xxhdpi</td><td>xxxhdpi</td></tr>
<tr><td>QVGA 240x320</td><td>HVGA 480*320</td><td>WVGA 800*480</td><td>720P 1280x720</td><td>1080P 1920x1080</td><td>1440x2560</td></tr>
<tr><td></td><td></td><td>FWVGA 480x854</td><td>1184x720</td><td></td><td></td></tr>
<tr><td></td><td></td><td>QHD 960*540</td><td></td><td></td><td></td></tr>
<tr><td></td></tr>
</table>
>常用手机的分辨率
iphone 4/4s 960*640 (3:2)
iphone5 1136*640
小米1 854*480(FWVGA)
小米2 1280*720