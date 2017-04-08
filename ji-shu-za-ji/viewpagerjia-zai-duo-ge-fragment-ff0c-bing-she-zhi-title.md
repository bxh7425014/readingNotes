2015-11-03 Viewpager加载多个Fragment，并设置Title

效果图如下：
![这里写图片描述](http://img.blog.csdn.net/20151103160658249)
# SyncHorizontalScrollView.java
```java
public class SyncHorizontalScrollView extends HorizontalScrollView {
  // Todo...
  private View view;
  private ImageView leftImage;// 左标图片
  private ImageView rightImage;// 右标图片
  private int windowWitdh = 0;// 屏幕宽度
  private Activity mContext;
  public void setSomeParam(View view, ImageView iv_nav_left,
  ImageView iv_nav_right, Activity context, int widthPixels) {
  this.view = view;
  this.mContext = context;
  this.leftImage = iv_nav_left;
  this.rightImage = iv_nav_right;
  this.windowWitdh = widthPixels;
  }
  // 显示和隐藏左右两边的箭头
  public void showAndHideArrow() {
  if (!mContext.isFinishing() && view != null) {
  this.measure(0, 0);
  if (windowWitdh >= this.getMeasuredWidth()) {
  leftImage.setVisibility(View.GONE);
  rightImage.setVisibility(View.GONE);
  } else {
  if (this.getLeft() == 0) {
  leftImage.setVisibility(View.GONE);
  rightImage.setVisibility(View.VISIBLE);
  } else if (this.getRight() == this.getMeasuredWidth()
  - windowWitdh) {
  leftImage.setVisibility(View.VISIBLE);
  rightImage.setVisibility(View.GONE);
  } else {
  leftImage.setVisibility(View.VISIBLE);
  rightImage.setVisibility(View.VISIBLE);
  }
  }
  }
  }
  @Override
  protected void onScrollChanged(int l, int t, int oldl, int oldt) {
  super.onScrollChanged(l, t, oldl, oldt);
  if (!mContext.isFinishing() && view != null && rightImage != null
  && leftImage != null) {
  if (view.getWidth() <= windowWitdh) {
  leftImage.setVisibility(View.GONE);
  rightImage.setVisibility(View.GONE);
  } else {
  if (l == 0) {
  leftImage.setVisibility(View.GONE);
  rightImage.setVisibility(View.VISIBLE);
  } else if (view.getWidth() - l == windowWitdh) {
  leftImage.setVisibility(View.VISIBLE);
  rightImage.setVisibility(View.GONE);
  } else {
  leftImage.setVisibility(View.VISIBLE);
  rightImage.setVisibility(View.VISIBLE);
  }
  }
  }
  }
}
```
# MainActivity.java
```java
public class MainActivity extends FragmentActivity {
  public static final String ARGUMENTS_NAME = "arg";
  private RelativeLayout rl_nav;
  private SyncHorizontalScrollView mHsv;
  private RadioGroup rg_nav_content;
  private ImageView iv_nav_indicator;
  private ImageView iv_nav_left;
  private ImageView iv_nav_right;
  private ViewPager mViewPager;
  private int indicatorWidth; // 导航条的标题宽度
  private final int NUM_OF_TITLEBAR = 3;
  private LayoutInflater mInflater;
  private TabFragmentPagerAdapter mAdapter;
  private int currentIndicatorLeft = 0;
  public static String[] tabTitle = { "标题1", "标题2", "标题3","标题4","标题5"};
  @Override
  protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);
  findViewById();
  initView();
  setListener();
  }
  private void findViewById() {
  rl_nav = (RelativeLayout) findViewById(R.id.rl_nav);
  mHsv = (SyncHorizontalScrollView) findViewById(R.id.mHsv);
  rg_nav_content = (RadioGroup) findViewById(R.id.rg_nav_content);
  iv_nav_indicator = (ImageView) findViewById(R.id.iv_nav_indicator);
  iv_nav_left = (ImageView) findViewById(R.id.iv_nav_left);
  iv_nav_right = (ImageView) findViewById(R.id.iv_nav_right);
  mViewPager = (ViewPager) findViewById(R.id.mViewPager);
  }
  private void initView() {
  DisplayMetrics dm = new DisplayMetrics();
  getWindowManager().getDefaultDisplay().getMetrics(dm);
  indicatorWidth = dm.widthPixels / NUM_OF_TITLEBAR;
  LayoutParams cursor_Params = iv_nav_indicator.getLayoutParams();
  cursor_Params.width = indicatorWidth;
  iv_nav_indicator.setLayoutParams(cursor_Params);
  mHsv.setSomeParam(rl_nav, iv_nav_left, iv_nav_right, this,
  dm.widthPixels);
  mInflater = (LayoutInflater) this
  .getSystemService(LAYOUT_INFLATER_SERVICE);
  initNavigationHSV();
  mAdapter = new TabFragmentPagerAdapter(getSupportFragmentManager());
  mViewPager.setAdapter(mAdapter);
  }
  private void initNavigationHSV() {
  rg_nav_content.removeAllViews();
  for (int i = 0; i < tabTitle.length; i++) {
  RadioButton rb = (RadioButton) mInflater.inflate(
  R.layout.nav_radiogroup_item, null);
  rb.setId(i);
  rb.setText(tabTitle[i]);
  rb.setLayoutParams(new LayoutParams(indicatorWidth,
  LayoutParams.MATCH_PARENT));
  rg_nav_content.addView(rb);
  }
  }
  public class TabFragmentPagerAdapter extends FragmentPagerAdapter {
  public TabFragmentPagerAdapter(FragmentManager fm) {
  super(fm);
  }
  @Override
  public Fragment getItem(int arg) {
  Fragment ft = null;
  switch (arg) {
  default:
  ft = new WebviewFrag();
  break;
  }
  return ft;
  }
  @Override
  public int getCount() {
  return tabTitle.length;
  }
  }
  private void setListener() {
  // 左右滑动屏幕时候触发
  mViewPager.setOnPageChangeListener(new OnPageChangeListener() {
  @Override
  public void onPageSelected(int position) {
  if (rg_nav_content != null
  && rg_nav_content.getChildCount() > position) {
  ((RadioButton) rg_nav_content.getChildAt(position))
  .performClick();
  }
  }
  @Override
  public void onPageScrolled(int arg0, float arg1, int arg2) {
  }
  @Override
  public void onPageScrollStateChanged(int arg0) {
  }
  });
  rg_nav_content
  .setOnCheckedChangeListener(new OnCheckedChangeListener() {
  @Override
  public void onCheckedChanged(RadioGroup group, int checkedId) {
  if (rg_nav_content.getChildAt(checkedId) != null) {
  TranslateAnimation animation = new TranslateAnimation(
  currentIndicatorLeft,
  ((RadioButton) rg_nav_content
  .getChildAt(checkedId)).getLeft(),
  0f, 0f);
  animation.setInterpolator(new LinearInterpolator());
  animation.setDuration(100);
  animation.setFillAfter(true);
  iv_nav_indicator.startAnimation(animation);
  mViewPager.setCurrentItem(checkedId);
  currentIndicatorLeft = ((RadioButton) rg_nav_content
  .getChildAt(checkedId)).getLeft();
  mHsv.smoothScrollTo(
  (checkedId > 1 ? ((RadioButton) rg_nav_content
  .getChildAt(checkedId)).getLeft()
  : 0)
  - ((RadioButton) rg_nav_content
  .getChildAt(2)).getLeft(),
  0);
  }
  }
  });
  }
}
```
# WebviewFrag.java
```java
public class WebviewFrag extends Fragment {
  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container,
  Bundle savedInstanceState) {
  View rootView = inflater.inflate(R.layout.fragment_common,
  container, false);
  WebView webView = (WebView) rootView.findViewById(R.id.webView);
  webView.loadUrl("http://www.baidu.com");
  initWebView(webView);
  return rootView;
  }
  private void initWebView(WebView wv) {
  wv.setWebViewClient(new WebViewClient() {
  public boolean shouldOverrideUrlLoading(WebView view, String url) {
  if (url.startsWith("file:") || url.startsWith("http:")
  || url.startsWith("https:")) {
  //view.loadUrl(url);
  return false;
  } else {
  return true;
  }
  }
  });
  }
}
```
# activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:context="${packageName}.${activityClass}" >
  <RelativeLayout
  android:id="@+id/rl_tab"
  android:layout_width="fill_parent"
  android:layout_height="wrap_content"
  android:background="#F2F2F2" >
  <com.example.viewpager_fragment_demo.SyncHorizontalScrollView
  android:id="@+id/mHsv"
  android:layout_width="fill_parent"
  android:layout_height="40dip"
  android:fadingEdge="none"
  android:scrollbars="none" >
  <RelativeLayout
  android:id="@+id/rl_nav"
  android:layout_width="fill_parent"
  android:layout_height="wrap_content"
  android:layout_gravity="top"
  android:background="#5AB0EB" >
  <RadioGroup
  android:id="@+id/rg_nav_content"
  android:layout_width="fill_parent"
  android:layout_height="38dip"
  android:layout_alignParentTop="true"
  android:background="#F2F2F2"
  android:orientation="horizontal" >
  </RadioGroup>
  <!-- 设置导航文字下方条标示线的颜色 -->
  <ImageView
  android:id="@+id/iv_nav_indicator"
  android:layout_width="1dip"
  android:layout_height="5dip"
  android:layout_alignParentBottom="true"
  android:background="#5AB0EB"
  android:contentDescription="@string/nav_desc"
  android:scaleType="matrix" />
  </RelativeLayout>
  </com.example.viewpager_fragment_demo.SyncHorizontalScrollView>
  <ImageView
  android:id="@+id/iv_nav_left"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:layout_alignParentLeft="true"
  android:layout_centerVertical="true"
  android:contentDescription="@string/nav_desc"
  android:paddingBottom="1dip"
  android:src="@drawable/iv_navagation_scroll_left"
  android:visibility="gone" >
  </ImageView>
  <ImageView
  android:id="@+id/iv_nav_right"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:layout_alignParentRight="true"
  android:layout_centerVertical="true"
  android:contentDescription="@string/nav_desc"
  android:paddingBottom="1dip"
  android:src="@drawable/iv_navagation_scroll_right"
  android:visibility="visible" >
  </ImageView>
  </RelativeLayout>
  <android.support.v4.view.ViewPager
  android:id="@+id/mViewPager"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:layout_alignParentBottom="true"
  android:layout_below="@id/rl_tab"
  android:layout_gravity="center"
  android:background="#ffffff"
  android:flipInterval="30"
  android:persistentDrawingCache="animation" />
</RelativeLayout>
```
# fragment_common.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical" >
  <WebView
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:id="@+id/webView"
  />
</LinearLayout>
```
# nav_radiogroup_item.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical" >
  <WebView
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:id="@+id/webView"
  />
</LinearLayout>
```
> 完整的代码请[点击此处访问](https://code.csdn.net/bxh7425014/csdn_blog_democode/tree/master/Viewpager_HorizontalScrollView)。