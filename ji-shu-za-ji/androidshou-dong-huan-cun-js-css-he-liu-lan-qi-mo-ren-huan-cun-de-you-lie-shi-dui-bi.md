2015-11-04 Android手动缓存js，css和浏览器默认缓存的优劣势对比

# 为什么用缓存
将html,js和css缓存到localStorage，可以减少Http请求，从而优化页面加载时间。
# 手动Web缓存
## 优势
- 可以缓存更多的内容到本地，包括大的图片。
## 劣势
- 需要设计良好的缓存策略，并且客户端和服务端都需要实现。
- 当版本出现重大更新增加客户端和服务器协作更新的复杂度。
- 对于跨域请求问题，用JSONP实现，需要服务器和客户端协商编写处理方法。
# WebView默认缓存
## 优势
- 设置方便，不用考虑交互问题。
- 可以设置不同的缓存策略，降低服务器的负载。
- 网上参考资料比较多，服务端可以通过标准配置来和客户端交互。客户端和服务器根据标准独立开发，不需商定协作策略。
## 劣势
- 缓存容量不能设置太大，耗费流量较多的图片缓存机制客户端无法控制。
# 手动加载Web缓存实战
- 以jquery mobile为例，在assets目录里面放入js,css,html资源文件。
![这里写图片描述](http://img2.tuicool.com/YBRfQni.jpg)
- 在写本地html的时候引入assert里的对应路径
```xml
<!DOCTYPE html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1">
<title> 标题 </title>
<link rel="stylesheet" type="text/css" href="file:///android_asset/css/jquery.mobile-1.4.2.min.css">
<script src="file:///android_asset/js/jquery-1.7.1.min.js"></script>
<script src="file:///android_asset/js/jquery.mobile-1.4.2.min.js"></script>
</head>
<body>
　<div data-role="page">
  　<div data-role="header">
  　　<h1>My Title</h1>
  　</div>
  　<div data-role="content">
  <ul data-role="listview" data-inset="true" >
  <li><a href="#">Acura</a></li>
  <li><a href="#">Audi</a></li>
  <li><a href="#">BMW</a></li>
  <li><a href="#">Cadillac</a></li>
  <li><a href="#">Ferrari</a></li>
  </ul>
  　</div>
　</div><!-- /page -->
</body>
</html>
```
- 在代码里访问页面
```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);
  WebView webview = (WebView)findViewById(R.id.webView1);
  WebSettings wv_setttig = webview.getSettings();
  wv_setttig.setJavaScriptEnabled(true);
  String url = "file:///android_asset/index.html";
  webview.loadUrl(url);
  }
}
```
- 最后效果如下:
![这里写图片描述](http://img0.tuicool.com/IRv6bi.jpg)
# 打开WebView内置缓存机制实战
当我们加载Html时候，会在我们data/应用package下生成database与cache两个文件夹:
我们请求的Url记录是保存在webviewCache.db里，而url的内容是保存在webviewCache文件夹下.
WebView中存在着两种缓存：网页数据缓存（存储打开过的页面及资源）、H5缓存（即AppCache）。
## 网页缓存
### 缓存构成
/data/data/package_name/cache/
/data/data/package_name/database/webview.db
/data/data/package_name/database/webviewCache.db
综合可以得知 webview 会将我们浏览过的网页url已经网页文件(css、图片、js等)保存到数据库表中
### 缓存模式(5种)
LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
LOAD_DEFAULT: 根据cache-control决定是否从网络上取数据。
LOAD_CACHE_NORMAL: API level 17中已经废弃, 从API level 11开始作用同LOAD_DEFAULT模式
LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。
如：www.taobao.com的cache-control为no-cache，在模式LOAD_DEFAULT下，无论如何都会从网络上取数据，如果没有网络，就会出现错误页面；在LOAD_CACHE_ELSE_NETWORK模式下，无论是否有网络，只要本地有缓存，都使用缓存。本地没有缓存时才从网络上获取。
www.360.com.cn的cache-control为max-age=60，在两种模式下都使用本地缓存数据。
> 总结：根据以上两种模式，建议缓存策略为，判断是否有网络，有的话，使用LOAD_DEFAULT，无网络时，使用LOAD_CACHE_ELSE_NETWORK。
### 设置WebView 缓存模式
```java
private void initWebView() {
  mWebView.getSettings().setJavaScriptEnabled(true);
  mWebView.getSettings().setRenderPriority(RenderPriority.HIGH);
  mWebView.getSettings().setCacheMode(WebSettings.LOAD_DEFAULT); //设置 缓存模式
  // 开启 DOM storage API 功能
  mWebView.getSettings().setDomStorageEnabled(true);
  //开启 database storage API 功能
  mWebView.getSettings().setDatabaseEnabled(true);
  String cacheDirPath = getFilesDir().getAbsolutePath()+APP_CACAHE_DIRNAME;
// String cacheDirPath = getCacheDir().getAbsolutePath()+Constant.APP_DB_DIRNAME;
  Log.i(TAG, "cacheDirPath="+cacheDirPath);
  //设置数据库缓存路径
  mWebView.getSettings().setDatabasePath(cacheDirPath);
  //设置 Application Caches 缓存目录
  mWebView.getSettings().setAppCachePath(cacheDirPath);
  //开启 Application Caches 功能
  mWebView.getSettings().setAppCacheEnabled(true);
  }
```
### 清除缓存
```java
/**
  * 清除WebView缓存
  */
  public void clearWebViewCache(){
  //清理Webview缓存数据库
  try {
  deleteDatabase("webview.db");
  deleteDatabase("webviewCache.db");
  } catch (Exception e) {
  e.printStackTrace();
  }
  //WebView 缓存文件
  File appCacheDir = new File(getFilesDir().getAbsolutePath()+APP_CACAHE_DIRNAME);
  Log.e(TAG, "appCacheDir path="+appCacheDir.getAbsolutePath());
  File webviewCacheDir = new File(getCacheDir().getAbsolutePath()+"/webviewCache");
  Log.e(TAG, "webviewCacheDir path="+webviewCacheDir.getAbsolutePath());
  //删除webview 缓存目录
  if(webviewCacheDir.exists()){
  deleteFile(webviewCacheDir);
  }
  //删除webview 缓存 缓存目录
  if(appCacheDir.exists()){
  deleteFile(appCacheDir);
  }
  }
```
### 完整代码
```java
package com.example.webviewtest;
import java.io.File;
import android.app.Activity;
import android.graphics.Bitmap;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.webkit.JsPromptResult;
import android.webkit.JsResult;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebSettings.RenderPriority;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.RelativeLayout;
import android.widget.TextView;
import android.widget.Toast;
public class MainActivity extends Activity {
  private static final String TAG = MainActivity.class.getSimpleName();
  private static final String APP_CACAHE_DIRNAME = "/webcache";
  private TextView tv_topbar_title;
  private RelativeLayout rl_loading;
  private WebView mWebView;
  private String url;
  @Override
  protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);
  //url:http://m.dianhua.cn/detail/31ccb426119d3c9eaa794df686c58636121d38bc?apikey=jFaWGVHdFVhekZYWTBWV1ZHSkZOVlJWY&app=com.yulore.yellowsdk_ios&uid=355136051337627
  url = "http://m.dianhua.cn/detail/31ccb426119d3c9eaa794df686c58636121d38bc?apikey=jFaWGVHdFVhekZYWTBWV1ZHSkZOVlJWY&app=com.yulore.yellowsdk_ios&uid=355136051337627";
  findView();
  }
  private void findView() {
  tv_topbar_title = (TextView) findViewById(R.id.tv_topbar_title);
  rl_loading = (RelativeLayout) findViewById(R.id.rl_loading);
  mWebView = (WebView) findViewById(R.id.mWebView);
  initWebView();
  mWebView.setWebViewClient(new WebViewClient() {
  @Override
  public void onLoadResource(WebView view, String url) {
  Log.i(TAG, "onLoadResource url="+url);
  super.onLoadResource(view, url);
  }
  @Override
  public boolean shouldOverrideUrlLoading(WebView webview, String url) {
  Log.i(TAG, "intercept url="+url);
  webview.loadUrl(url);
  return true;
  }
  @Override
  public void onPageStarted(WebView view, String url, Bitmap favicon) {
  Log.e(TAG, "onPageStarted");
  rl_loading.setVisibility(View.VISIBLE); // 显示加载界面
  }
  @Override
  public void onPageFinished(WebView view, String url) {
  String title = view.getTitle();
  Log.e(TAG, "onPageFinished WebView title=" + title);
  tv_topbar_title.setText(title);
  tv_topbar_title.setVisibility(View.VISIBLE);
  rl_loading.setVisibility(View.GONE); // 隐藏加载界面
  }
  @Override
  public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
  rl_loading.setVisibility(View.GONE); // 隐藏加载界面
  Toast.makeText(getApplicationContext(), "",
  Toast.LENGTH_LONG).show();
  }
  });
  mWebView.setWebChromeClient(new WebChromeClient() {
  @Override
  public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
  Log.e(TAG, "onJsAlert " + message);
  Toast.makeText(getApplicationContext(), message, Toast.LENGTH_SHORT).show();
  result.confirm();
  return true;
  }
  @Override
  public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
  Log.e(TAG, "onJsConfirm " + message);
  return super.onJsConfirm(view, url, message, result);
  }
  @Override
  public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
  Log.e(TAG, "onJsPrompt " + url);
  return super.onJsPrompt(view, url, message, defaultValue, result);
  }
  });
  mWebView.loadUrl(url);
  }
  private void initWebView() {
  mWebView.getSettings().setJavaScriptEnabled(true);
  mWebView.getSettings().setRenderPriority(RenderPriority.HIGH);
  mWebView.getSettings().setCacheMode(WebSettings.LOAD_DEFAULT); //设置 缓存模式
  // 开启 DOM storage API 功能
  mWebView.getSettings().setDomStorageEnabled(true);
  //开启 database storage API 功能
  mWebView.getSettings().setDatabaseEnabled(true);
  String cacheDirPath = getFilesDir().getAbsolutePath()+APP_CACAHE_DIRNAME;
// String cacheDirPath = getCacheDir().getAbsolutePath()+Constant.APP_DB_DIRNAME;
  Log.i(TAG, "cacheDirPath="+cacheDirPath);
  //设置数据库缓存路径
  mWebView.getSettings().setDatabasePath(cacheDirPath);
  //设置 Application Caches 缓存目录
  mWebView.getSettings().setAppCachePath(cacheDirPath);
  //开启 Application Caches 功能
  mWebView.getSettings().setAppCacheEnabled(true);
  }
  /**
  * 清除WebView缓存
  */
  public void clearWebViewCache(){
  //清理Webview缓存数据库
  try {
  deleteDatabase("webview.db");
  deleteDatabase("webviewCache.db");
  } catch (Exception e) {
  e.printStackTrace();
  }
  //WebView 缓存文件
  File appCacheDir = new File(getFilesDir().getAbsolutePath()+APP_CACAHE_DIRNAME);
  Log.e(TAG, "appCacheDir path="+appCacheDir.getAbsolutePath());
  File webviewCacheDir = new File(getCacheDir().getAbsolutePath()+"/webviewCache");
  Log.e(TAG, "webviewCacheDir path="+webviewCacheDir.getAbsolutePath());
  //删除webview 缓存目录
  if(webviewCacheDir.exists()){
  deleteFile(webviewCacheDir);
  }
  //删除webview 缓存 缓存目录
  if(appCacheDir.exists()){
  deleteFile(appCacheDir);
  }
  }
  /**
  * 递归删除 文件/文件夹
  *
  * @param file
  */
  public void deleteFile(File file) {
  Log.i(TAG, "delete file path=" + file.getAbsolutePath());
  if (file.exists()) {
  if (file.isFile()) {
  file.delete();
  } else if (file.isDirectory()) {
  File files[] = file.listFiles();
  for (int i = 0; i < files.length; i++) {
  deleteFile(files[i]);
  }
  }
  file.delete();
  } else {
  Log.e(TAG, "delete file no exists " + file.getAbsolutePath());
  }
  }
}
```