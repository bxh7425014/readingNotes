[转]自定义LyricView实现歌词显示控件

[原文地址](http://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650236950&idx=1&sn=d51e6420df7f7533bc81cfe98541c3c0&scene=1&srcid=0902EjpPrGAyIOBJAIOCNg11#rd) | [源码地址](https://github.com/bxh7425014/LyricViewDemo)

> 注：原文的代码阅读和拷贝起来不太方便，我已经摘录出来。

# 前言
通过自定义View，实现的进阶版LyricView ，能够实现歌词滑动查看，当前播放位置高亮显示，滑动到指定位置并播放等等，大致和网易云音乐的歌词显示效果一样。

# 歌词文件的组成

```
[ti:一个人的北京]
[ar:好妹妹乐队]
[al:南北]
[by:]
[offset:0]
[00:00.10]一个人的北京 - 好妹妹乐队
[00:00.20]词：秦昊
[00:00.30]曲：秦昊
[00:00.40]
[00:30.16]你有多久没有看到 满天的繁星
[00:37.34]城市夜晚虚伪的光明 遮住你的眼睛
......
[04:48.87]离开了这里 在晴朗的天气
[04:55.08]
[04:56.27]让我拥抱你 在晴朗的天气
```

歌词文件（*.lrc）都是以一个标准来进行制作的。
```
[ti:  标题
[ar:  歌手
[al:  专辑
[by: 制作
[offset: 时间偏移量
[mm:ss.ms] 歌词信息:由 开始时间（分：秒.毫秒）和 歌词内容 两部分组成
```

# 解析歌词文件
首先获取*.lrc歌词文件的二进制流 InputStream，再又转换成字符流（注意：转化成字符流的时候需要选择编码，比如QQ音乐的歌词文件需要用”GBK”解码）。

- 准备两个类主要用于歌词解析结果的缓存：LyricInfo和 LineInfo（）：

LyricInfo 歌词信息：包含标题、歌手、专辑等等
```
class LyricInfo {
    List<LineInfo> song_lines; 
    String song_artist;//歌手
    String song_title;//标题
    String song_album;//专辑
    long song_offset;//偏移量
}
```

LineInfo 歌词行信息：包含行开始时间和歌词行内容
``` 
class LineInfo {
    String content;//歌词内容
    long start;//开始时间
}
```

解析歌词文件源码：
```
/**
 * 初始化歌词信息
 * @param inputStream  歌词文件的流信息
 * */
private void setupLyricResource(InputStream inputStream, String charsetName) {
    if(inputStream != null) {
        try {
            LyricInfo lyricInfo = new LyricInfo();
            lyricInfo.song_lines = new ArrayList<>();
            InputStreamReader inputStreamReader = new InputStreamReader(inputStream, charsetName);
            BufferedReader reader = new BufferedReader(inputStreamReader);
            String line = null;
            while((line = reader.readLine()) != null) {
                analyzeLyric(lyricInfo, line);
            }
            reader.close();
            inputStream.close();
            inputStreamReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    } else {
        // 暂无歌词
    }
}

/**
 * 逐行解析歌词内容
 * */
private void analyzeLyric(LyricInfo lyricInfo, String line) {
    int index = line.lastIndexOf("]");
    if(line != null && line.startsWith("[offset:")) {
        // 时间偏移量
        String string = line.substring(8, index).trim();
        lyricInfo.song_offset = Long.parseLong(string);
        return;
    }
    if(line != null && line.startsWith("[ti:")) {
        // title 标题
        String string = line.substring(4, index).trim();
        lyricInfo.song_title = string;
        return;
    }
    if(line != null && line.startsWith("[ar:")) {
        // artist 作者
        String string = line.substring(4, index).trim();
        lyricInfo.song_artist = string;
        return;
    }
    if(line != null && line.startsWith("[al:")) {
        // album 所属专辑
        String string = line.substring(4, index).trim();
        lyricInfo.song_album = string;
        return;
    }
    if(line != null && line.startsWith("[by:")) {
        return;
    }
    if(line != null && index == 9 && line.trim().length() > 10) {
        // 歌词内容
        LineInfo lineInfo = new LineInfo();
        lineInfo.content = line.substring(10, line.length());
        lineInfo.start = measureStartTimeMillis(line.substring(0, 10));
        lyricInfo.song_lines.add(lineInfo);
    }
}

/**
 * 从字符串中获得时间值
 * */
private long measureStartTimeMillis(String str) {
    long minute = Long.parseLong(str.substring(1, 3));
    long second = Long.parseLong(str.substring(4, 6));
    long millisecond = Long.parseLong(str.substring(7, 9));
    return millisecond + second * 1000 + minute * 60 * 1000;
}
```

# 验证解析效果

完成歌词解析，接下来就是验证歌词解析的一个实际效果的时候了：
```
File file = new File(Constant.lyricPath + "一个人的北京 - 好妹妹乐队.lrc");
if (file != null && file.exists()) {
    try {
        setupLyricResource(new FileInputStream(file), "GBK");
        StringBuffer stringBuffer = new StringBuffer();
        if(lyricInfo != null && lyricInfo.song_lines != null) {
            int size = lyricInfo.song_lines.size();
            for (int i = 0; i < size; i ++) {
                stringBuffer.append(lyricInfo.song_lines.get(i).content + "\n");
            }
            text.setText(stringBuffer.toString());
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```

效果图：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-e5117c3d046ecf39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就这样，一个简单的歌词显示功能也就实现了。 但是，如何才能够让自己写的音乐播放器在歌词显示模块能够显得高大上，并且能够像很多当前应用市场上流行的音乐播放器那样，实现当前播放高亮显示、歌词回弹效果、歌词淡入淡出效果以及滑动歌词快速播放等功能呢？ 请接着往下读…..

上面有提及到在早些前我有用 ScrollView 嵌套 TextView 的方式实现过自定义 LyricView，但是，由于体验效果和功能拓展上的不足，我并没有公开分享。既然通过 ScrollView 嵌套 TextView 的方式 不能满足 我们的设计需求，那是不是能够通过 自定义View 的方式实现 LyricView？既有如 TextView 那样的 LineHeigh(行高)、LineCount(总行数) 的概念，也有如 ScrollView 那样的 ScrollY(Y方向的偏移量) 的概念。那是必须的，说干就干。

# LyricView实现
解析*.lrc歌词文件，生成歌词集合列表，获得行总数

上面我已经讲过解析歌词了，而在 LyricView 中，我们需要做的是将逐行解析出来的歌词信息添加到 集合mLyricInfo 中，而 总行数mLineCount 也就等于 List集合 的大小 mLineCount = mLyricInfo.song_lines.size()。

计算歌词单行高度，获得歌词绘制区域总高度

写过 自定义View 的朋友应该都会知道，在 自定义View 中如果涉及文字的绘制，为了能够精准的绘制文字的位置，我们需要获取需要绘制的文字的矩形区域，通过画笔 Paint 的 getTextBounds(String text, int start, int end, Rect bounds)方法 则可以帮助我们轻松获得一个需要绘制文字的一个矩形。

当然，绘制文字矩形区域的大小还与文字的大小相关，我们还可以通过画笔 Paint 的 setTextSize(float textSize)方法 设置绘制文字大小，也就是常说的 TextSize。

在 LyricView 中，行高可不仅仅就只是文字矩形区域的高度，行高还包括两行之间的行间距，如下图所示。既然歌词总行数和歌词单行高度我们都已取得，那么获取歌词绘制区域的总高度也就so easy了：
```
/**
 * 计算行高度
 * */
private void measureLineHeight() {
    Rect lineBound = new Rect();
    mTextPaint.getTextBounds(mDefaultHint, 0, mDefaultHint.length(), lineBound);
    mLineHeight = lineBound.height() + mLineSpace;
}
```

计算行高度

定义 scrollY，并通过当前歌曲播放进度的时间戳计算 scrollY

既然 LyricView 能够实现滑动功能，那么引入 scrollY值 记录滑动偏移量，并控制视图绘制效果也就顺理成章。 需要明确一点，当偏移量 scrollY 的值为零的时候，歌词的首行将显示在整个 LyricView 的正中间 。

我们知道每一句歌词中都包含着开始时间，而我们也就可以通过当前歌曲播放进度匹配当前播放的行数 mCurrentPlayLine，并通过当前播放所在行，计算偏移量 scrollY 的值，控制歌词播放滚动和当前播放位置的高亮显示。
```
for(int i = 0, size = mLineCount; i < size; i ++) {
    LineInfo lineInfo = mLyricInfo.song_lines.get(i);
    if(lineInfo != null && lineInfo.start > time) {
        position = i;
        break;
    }
    if(i == mLineCount - 1) {
        position = mLineCount;
    }
}
```
匹配当前播放行数 mCurrentPlayLine
```
/**
 * Input current showing line to measure the view's current scroll Y
 * @param line  当前指定行号
 * */
private float measureCurrentScrollY(int line) {
    return (line - 1) * mLineHeight;
}
```
计算偏移量scrollY

理论基础已经实现，初步尝试绘图 onDraw：
```
for(int i = 0, size = mLineCount; i < size; i ++) {
    float x = getMeasuredWidth() * 0.5f;
    float y = getMeasuredHeight() * 0.5f + (i + 0.5f) * mLineHeight - 6 - mLineSpace * 0.5f - mScrollY;
    if(y + mLineHeight * 0.5f < 0) {
        continue;
    }
    if(y - mLineHeight * 0.5f > getMeasuredHeight()) {
        break;
    }
    if(i == mCurrentPlayLine - 1) {
        mTextPaint.setColor(mHighLightColor);
    } else {
        if(mIndicatorShow && i == mCurrentShowLine - 1) {
            mTextPaint.setColor(mCurrentShowColor);
        }else {
            mTextPaint.setColor(mDefaultColor);
        }
    }
    if(y > getMeasuredHeight() - mShaderWidth || y < mShaderWidth) {
        if(y < mShaderWidth) {
            mTextPaint.setAlpha(26 + (int) (23000.0f * y / mShaderWidth * 0.01f));
        } else {
            mTextPaint.setAlpha(26 + (int) (23000.0f * (getMeasuredHeight() - y) / mShaderWidth * 0.01f));
        }
    } else {
        mTextPaint.setAlpha(255);
    }
    canvas.drawText(mLyricInfo.song_lines.get(i).content, x, y, mTextPaint);
}
```

Bingo ! 歌词确实能够在屏幕上绘制出来，细心的朋友也许会发现其中的几个特别的地方，分别是当前播放位置高亮显示和歌词淡入淡出效果实现的代码：

```
//实现当前位置高亮显示
if(i == mCurrentPlayLine - 1) {
    mTextPaint.setColor(mHighLightColor);
}
//歌词淡入淡出效果实现
if(y > getMeasuredHeight() - mShaderWidth || y < mShaderWidth) {
    if(y < mShaderWidth) {
        mTextPaint.setAlpha(26 + (int) (23000.0f * y / mShaderWidth * 0.01f));
    } else {
        mTextPaint.setAlpha(26 + (int) (23000.0f * (getMeasuredHeight() - y) / mShaderWidth * 0.01f));
    }
} else {
    mTextPaint.setAlpha(255);
}
```
但是，仅仅只是实现显示功能，并且超出范围的歌词内容还不能通过滑动来查看。哈哈~ 别着急啊，骚年，坐下来和我凉茶，听我细细道来。

重写 onTouchEvent，实现歌词滑动查看，并实现 overScroll 回弹效果

仅仅需要实现滑动视图的功能的话，说实话，非常简单，只需要记录 ACTION_DOWN 时的 y值，并比较 ACTION_MOVE 过程中的 y值 计算两者的差值，生成新的偏移量 scrollY，再刷新视图，就可以搞定 !

要是就这么简简单单了事的话，怎么也不符合我个人对完美设计的要求。要是我们无限滑动的话，整个歌词内容区域就会滑动出我们的可视区域，也就是常说的 overScroll，如果不加以限制将会是一种非常差的用户体验。

当然，不同的开发对解决这个问题有不同的方法，有些播放器的歌词显示控件，当滑动事件出现 overScroll 时，将不再视图继续滑动。当然，也有当滑动事件出现 overScroll 时，视图依旧能够继续滑动，但与正常滑动时有所区别，这个时候的滑动会有一种 阻尼效果，也就是实际滑动距离和视图的滚动距离并不相等，而且随着 overScroll 的值越大，阻力越大，滑动越艰难，并在用户手指离开屏幕后回到 overScrol l的值为零的位置。当然，我本人更喜欢后者的用户体验，为了实现这个功能，那么就必须要在重写 onTouchEvent 的方法中”做点手脚”了。
```
/**
 * 手势移动执行事件
 * @param event
 * */
private void actionMove(MotionEvent event) {
    if(scrollable()) {
        final VelocityTracker tracker = mVelocityTracker;
        tracker.computeCurrentVelocity(1000, maximumFlingVelocity);
        float scrollY = mLastScrollY + mDownY - event.getY();   // 102  -2  58  42
        float value01 = scrollY - (mLineCount * mLineHeight * 0.5f);   // 52  -52  8  -8
        float value02 = ((Math.abs(value01) - (mLineCount * mLineHeight * 0.5f)));   // 2  2  -42  -42
        mScrollY = value02 > 0 ? scrollY - (measureDampingDistance(value02) * value01 / Math.abs(value01)) : scrollY;   //   value01 / Math.abs(value01)  控制滑动方向
        mVelocity = tracker.getYVelocity();
        measureCurrentLine();
    }
}
```
ACTION_MOVE
```
/**
 * 计算阻尼效果的大小
 * */
private final int mMaxDampingDistance = 360;
private float measureDampingDistance(float value02) {
    return value02 > mMaxDampingDistance ? (mMaxDampingDistance * 0.6f + (value02 - mMaxDampingDistance) * 0.72f) : value02 * 0.6f;
}
```

阻尼大小计算

通过我一次一次对代码的细化，只要这么简单的两个方法，就完成了滑动时偏移量 scrollY 的计算，包括 overScroll 和 非overScroll，是的，只要这么两个方法。

到了这一步，歌词的显示、滑动查看都已经完成。但这还没完，我是不是还说过我的 LyricView 能够实现像网易云音乐歌词显示控件那样的指示器效果，哈哈哈 ~ 对于我这个完美主义者而言，这个功能必须实现。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-35761fd33ee31266.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

歌词指示器效果图

实现歌词指示器效果，”屌丝”蜕变”高富帅”

其实，指示器效果实现起来也不是很难，其实指示器左侧的按钮完全可以用绘制 Bitmap 的方式其实现，但是，考虑到 LyricView 的灵活性，同时，我们程序猿不都是能够用代码绘制的决不在工程中添加图片的嘛 ！更何况就一个简单的播放按钮，随便画画，哈哈 ~ 至于，右侧的时间指示，则是通过当前偏移量 scrollY 的值计算得来的当前控件正中间位置显示歌词的开始时间。

```

/**
 * 绘制左侧的播放按钮
 * @param canvas
 * */
private void drawPlayer(Canvas canvas) {
    mBtnBound = new Rect(mDefaultMargin, (int) (getMeasuredHeight() * 0.5f - mBtnWidth * 0.5f), mBtnWidth + mDefaultMargin, (int) (getMeasuredHeight() * 0.5f + mBtnWidth * 0.5f));

    Path path = new Path();
    float radio = mBtnBound.width() * 0.3f;
    float value = (float) Math.sqrt(Math.pow(radio, 2) - Math.pow(radio * 0.5f, 2));
    path.moveTo(mBtnBound.centerX() - radio * 0.5f, mBtnBound.centerY() - value);
    path.lineTo(mBtnBound.centerX() - radio * 0.5f, mBtnBound.centerY() + value);
    path.lineTo(mBtnBound.centerX() + radio, mBtnBound.centerY());
    path.lineTo(mBtnBound.centerX() - radio * 0.5f, mBtnBound.centerY() - value);
    mBtnPaint.setAlpha(128);
    canvas.drawPath(path, mBtnPaint);  // 绘制播放按钮的三角形
    canvas.drawCircle(mBtnBound.centerX(), mBtnBound.centerY(), mBtnBound. width() * 0.48f, mBtnPaint);  // 绘制圆环
}
```

绘制指示器左侧播放按钮
```
/**
 * 绘制指示器
 * @param canvas
 * */
private void drawIndicator(Canvas canvas) {
    mIndicatorPaint.setColor(mIndicatorColor);
    mIndicatorPaint.setAlpha(128);
    mIndicatorPaint.setStyle(Paint.Style.FILL);
    canvas.drawText(measureCurrentTime(), getMeasuredWidth() - mTimerBound.width(), (getMeasuredHeight() + mTimerBound.height() - 6) * 0.5f, mIndicatorPaint);

    Path path = new Path();
    mIndicatorPaint.setStrokeWidth(2.0f);
    mIndicatorPaint.setStyle(Paint.Style.STROKE);
    mIndicatorPaint.setPathEffect(new DashPathEffect(new float[]{20, 10}, 0));
    path.moveTo(mPlayable ? mBtnBound.right + 24 : 24 , getMeasuredHeight() * 0.5f);
    path.lineTo(getMeasuredWidth() - mTimerBound.width() - mTimerBound.width() - 36, getMeasuredHeight() * 0.5f);
    canvas.drawPath(path , mIndicatorPaint);
}
```

绘制指示器分割线和时间

既然设计播放按钮，当然播放按钮就要实现点击事件啊：
```
/**
 * 判断当前点击事件是否落在播放按钮触摸区域范围内
 * @param event  触摸事件
 * */
private boolean clickPlayer(MotionEvent event) {
    if(mBtnBound != null &&  mDownX > (mBtnBound.left - mDefaultMargin) && mDownX < (mBtnBound.right + mDefaultMargin) && mDownY > (mBtnBound.top - mDefaultMargin) && mDownY < (mBtnBound.bottom + mDefaultMargin)) {
        float upX = event.getX();   float upY = event.getY();
        return upX > (mBtnBound.left - mDefaultMargin) && upX < (mBtnBound.right + mDefaultMargin) && upY > (mBtnBound.top - mDefaultMargin) && upY < (mBtnBound.bottom + mDefaultMargin);
    }
    return false;
}
```

 播放按钮点击位置判断
```
if(mIndicatorShow && clickPlayer(event)) {
    if(mCurrentShowLine != mCurrentPlayLine) {
        mIndicatorShow = false;
        if(mClickListener != null) {
            mClickListener.onPlayerClicked(
                mLyricInfo.song_lines.get(mCurrentShowLine - 1).start,
                mLyricInfo.song_lines.get(mCurrentShowLine - 1).content);
        }
    }
}
```

到这一步，我们的自定义 LyricView 设计介绍也就告一段落咯 ！ 当然，功能远不止这些，还有 设置字体大小、设置行间距 以及 结合速度追踪器实现滑行效果 等等。所谓”授人以鱼不如授人以渔”，我想要和大家分享的是我的一个设计思路，大家可以根据需求设计不通的功能，因此在这里我也不做过多介绍，对小阿飞的 LyricView 感兴趣的朋友可以去我的gitHub下载研究：
https://github.com/WuLiFei/LyricViewDemo

# 效果图

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-58fc721f947ec79a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
overScroll效果展示

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-4bc6529765d6c09a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

字体颜色设置效果展示


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-b3bc2e49587ab941.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
字体大小设置效果展示


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-7f76f6a69874a48a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
行间距设置效果展示


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-2eae3855575284d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
指示器和播放按钮效果展示
