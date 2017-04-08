2015-12-08 一个功能引导页面的实现思路（效果参考：美丽说app）

# 原型
美丽说app的首页引导效果图如下：
![美丽说首页引导](http://img.blog.csdn.net/20151208175835020)
下载美丽说的apk，解压后，找到切图如下：
![这里写图片描述](http://img.blog.csdn.net/20151208180409322)
> 可以看到，由于切图右下角留出白色透明圆圈，所以有了上面的效果。
# 进一步思考
由于android屏幕尺寸的碎片化，所以如果我们要做一张固定的切图，把透过来的部分留在固定位置并不是一个好的解决方案。所以可以考虑把“我”上面的圆圈用代码来绘制，如果要给让某一个控件透过压黑的背景显示出来，必须要精确控制背景要透明的效果。
# 实施
## 获取控件的位置
关键代码：
```java
ImageView t = (ImageView)findViewById(R.id.l);
  t.getLocationOnScreen(location);
  int x = location[0];
  int y = location[1];
  System.out.println("x:" + x + "y:" + y);
  System.out.println("图片各个角Left：" + t.getLeft() + "Right：" + t.getRight()
  + "Top：" + t.getTop() + "Bottom：" + t.getBottom());
```
## 制作屏幕宽高的图片
```java
  /**
  * 生成对应颜色的全屏图像
  * @param context
  * @param color
  * @return
  */
  public static Bitmap createColorWallpaer(Context context, int color) {
  WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
  Point outSize = new Point();
  wm.getDefaultDisplay().getSize(outSize);
  Bitmap bitmap = Bitmap.createBitmap(outSize.x, outSize.y, Bitmap.Config.ARGB_8888);
  if (bitmap != null) {
  bitmap.eraseColor(color);
  }
  return bitmap;
  }
```
> outSize就是屏幕的尺寸。
## 获取状态栏高度
因为制作背景图的时候，不要把状态栏算在内，所以制作背景图时，需要把状态栏的尺寸高度刨掉。
```java
Rect frame = new Rect();
getWindow().getDecorView().getWindowVisibleDisplayFrame(frame);
int statusBarHeight = frame.top;
```
> decorView是window中的最顶层view，可以从window中获取到decorView，然后decorView有个getWindowVisibleDisplayFrame方法可以获取到程序显示的区域，包括标题栏，但不包括状态栏。
于是，我们就可以算出状态栏的高度了。
# 生成我们需要的Bitmap
```java
  /**
  * 创建引导图像，在指定位置显示一个圆圈和圆角矩形
  * @param context
  * @param color
  * @return
  */
  public static Bitmap createTipsWallpaer(Context context, int color) {
  Bitmap bitmap = createColorWallpaer(context, color);
  // 创建画笔
  Paint p = new Paint();
  p.setColor(Color.TRANSPARENT);
  //设置图像的叠加模式
  p.setXfermode(new PorterDuffXfermode(Mode.SRC));
  Canvas canvas = new Canvas(bitmap);
  canvas.drawCircle(150, 150, 100, p);// 小圆
  //画圆角矩形
  p.setStyle(Paint.Style.FILL);//充满
  p.setAntiAlias(true);// 设置画笔的锯齿效果
  canvas.drawText("画圆角矩形:", 10, 260, p);
  RectF oval3 = new RectF(80, 260, 200, 300);// 设置个新的长方形
  canvas.drawRoundRect(oval3, 10, 10, p);//第二个参数是x半径，第三个参数是y半径
  try {
  saveMyBitmap("yiyixiaozhi", bitmap);
  } catch (IOException e) {
  // TODO Auto-generated catch block
  e.printStackTrace();
  }
  return bitmap;
  }
  /**
  * 保存Bitmap为本地文件
  * @param bitName
  * @param bmp
  * @throws IOException
  */
  public static void saveMyBitmap(String bitName, Bitmap bmp) throws IOException {
  File f = new File("/sdcard/Note/" + bitName + ".png");
  f.createNewFile();
  FileOutputStream fOut = null;
  try {
  fOut = new FileOutputStream(f);
  } catch (FileNotFoundException e) {
  e.printStackTrace();
  }
  bmp.compress(Bitmap.CompressFormat.PNG, 100, fOut);
  try {
  fOut.flush();
  } catch (IOException e) {
  e.printStackTrace();
  }
  try {
  fOut.close();
  } catch (IOException e) {
  e.printStackTrace();
  }
  }
```
下来我们用picasa打开本地保存的文件，成功了，效果如下：
![这里写图片描述](http://img.blog.csdn.net/20151208183601211)
> 注意，如果不对paint应用Mode.SRC（显示上层图像），是无法显示出来透明的部分的，截图如下：
> ![这里写图片描述](http://img.blog.csdn.net/20151208183837311)
如有兴趣进一步探讨，欢迎订阅我的微信公众号（yiyixiaozhi）留言给我。或者在此博客下方进行评论。
![这里写图片描述](http://img.blog.csdn.net/20151208184152633)
参考网址：
- [Android利用canvas画各种图形(点、直线、弧、圆、椭圆、文字、矩形、多边形、曲线、圆角矩形) ](http://blog.sina.com.cn/s/blog_4cd5d2bb0101g2la.html)
- [Android得到控件在屏幕中的坐标](http://blog.csdn.net/sjf0115/article/details/7306284)
- [Android bitmap图片处理](http://blog.csdn.net/dahuaishu2010_/article/details/28622417)
- [android中Bitmap对象怎么保存为文件?](http://blog.csdn.net/zhzhyang0313/article/details/6742473)
- [android 状态栏、标题栏、屏幕高度](http://xqjay19910131-yahoo-cn.iteye.com/blog/1435249)