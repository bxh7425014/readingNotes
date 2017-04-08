2015-10-30 Android Drawable图像转换.md
Drawable转bitmap，bitmap转png文件的方法
```java
public static Bitmap drawableToBitmap(Drawable drawable) {
  Bitmap bitmap = Bitmap
  . createBitmap(
  drawable.getIntrinsicWidth(),
  drawable.getIntrinsicHeight(),
  drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888
  : Bitmap.Config.RGB_565 );
  Canvas canvas = new Canvas(bitmap);
  // canvas.setBitmap(bitmap);
  drawable.setBounds(0, 0, drawable.getIntrinsicWidth(),
  drawable.getIntrinsicHeight());
  drawable.draw(canvas);
  return bitmap;
  }
  public static void saveMyBitmap (String bitName, Bitmap bmp) throws IOException {
  File f = new File("/data/data/com.test.yyxz/databases/" + bitName + ".png");
  f.createNewFile();
  FileOutputStream fOut = null;
  try {
  fOut = new FileOutputStream(f);
  } catch (FileNotFoundException e) {
  e.printStackTrace();
  }
  bmp.compress(Bitmap.CompressFormat. PNG, 100, fOut);
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