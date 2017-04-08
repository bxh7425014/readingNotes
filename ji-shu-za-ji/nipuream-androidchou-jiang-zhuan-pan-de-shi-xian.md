[Nipuream]-Android抽奖转盘的实现
[「原文链接」](http://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650237117&idx=1&sn=0fb217e06519c012402b55de34d19c17&chksm=886399d2bf1410c4c80da4a60c5944e1215c43cd9dd64022f50442ac9b9d312029202e9a21d3&scene=1&srcid=0918fap1E5f1ycHCowjDVS0n#rd)

Nipuream 的[博客地址](http://blog.csdn.net/yanghuinipurean)
# 序言
效果如下：
![点击Go按钮自动启动+自动滚动](http://upload-images.jianshu.io/upload_images/1124873-f7ef18870bb0749f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现的效果还不错，因为是模拟器加录制，画面可能会有些卡顿，真机其实蛮顺畅的，下面简单的讲讲实现的步骤。

# 实现
## 绘制
首先第一个我们要它给画出来，但是要注意的就是Android所对应的坐标系的问题。

![坐标系](http://upload-images.jianshu.io/upload_images/1124873-0d34f51368fee008.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-fd7120dfe51f6905.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中有两个地方需要注意下，**第一个**就是画弧的地方.**第一个角度是起始角度，第二个是弧的角度**，并不是结束的角度，所以是**固定值60**。**第二个地方**就是计算具体的 **x,y的值** 的时候要 **根据弧度去计算*，不能根据角度。

## 使用属性动画旋转

如果用 **SurfaceView** 去进行重绘旋转存在一些问题，比如旋转的角度不好控制，旋转的速度不好控制。但是用属性动画，这个问题就很好解决了。

![](http://upload-images.jianshu.io/upload_images/1124873-3d7e4e74ce22457f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用动画最重要的就是，如何计算出结束动画后的位置，那么把最终旋转的总角度%360°就得到最后一圈实际旋转的角度，再除以60就得到了到底选择了几个位置，因为一个位置占据60°，这应该不难理解。

![](http://upload-images.jianshu.io/upload_images/1124873-7e0e367e42acf6fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是问题又来了，Android所对应的坐标系，0的位置应该是最底下，而指针的位置是在最上面，所以，我们结合上面的坐标系来看，还需要处理下，如上面的代码所示。

## 处理手势
触摸事件的处理，最后到底允不允许转盘随手势滑动呢？其实貌似做成这样也就可以了，但是最后还是实现了下，用到了 **GestureDetector** 和 **Scroller** 这个类。其实做法有很多，首先获取我们的滑动的距离，**Math.sqrt(dx \* dx + dy * dy)**，然后无非就是把这个距离转换成我们需要的角度，你可以把这个距离当作我们的周长来处理，也可以把这个距离当作我们总的旋转的角度来处理。之后就是随着时间的流逝，不断的刷新我们的界面了。

![](http://upload-images.jianshu.io/upload_images/1124873-454c21ad168c0542.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-1619d815f9ff6be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 处理剩余问题

还存在个问题，如果没有手势去操作转盘，那我们很容易判断它所旋转的角度，但是有手势的参与，我们很容易旋转到转盘中两个分片中间的位置，那么，我们在让它旋转之前，要简单处理下，避免这种事情发生。

```
//TODO 为了每次都能旋转到转盘的中间位置
int offRotate = DesRotate % 360 % 60;
DesRotate -= offRotate;
DesRotate += 30;
```

这样不管手势怎么操作，我最终都是旋转到分片的中间位置了。

[代码地址](https://github.com/Nipuream/LuckPan)
[apk地址](https://github.com/Nipuream/LuckPan/raw/master/luck_pan.apk)