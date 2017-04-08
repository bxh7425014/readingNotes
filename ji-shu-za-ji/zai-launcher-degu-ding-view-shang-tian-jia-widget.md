2015-10-30 在Launcher的固定View上添加Widget_墨迹天气

首先感谢thl789博客中关于widget的解释，看后受益很多。有益的文章链接如下：
[Android AppWidget系统框架](http://blog.csdn.net/thl789/article/details/7879257)
[Android中选取并绑定AppWidget](http://blog.csdn.net/thl789/article/details/7880650)
[Android中AppWidget的分析与应用：AppWidgetProvider](http://blog.csdn.net/thl789/article/details/7887968)
[Android中Launcher对于AppWidget处理的分析：AppWidgetHost角色](http://blog.csdn.net/thl789/article/details/7893292)
[Android中RemoteViews的实现](http://blog.csdn.net/thl789/article/details/7893908)
下面是我的内容：
# 获取系统已安装的widget列表
测试代码：
```java
// Test code，获取系统已安装的Widget列表。
List<AppWidgetProviderInfo> installed = Launcher.mAppWidgetManager.getInstalledProviders();
for(int index = 0; index < installed.size(); index ++) {
  ComponentName cn = installed.get(index).provider;
  Log.i("test", "" + cn.getPackageName() + ", " + cn.getClassName());
}
```
打印log示例如下：
```
Line 1093: 08-23 12:54:39.469 I/test ( 5772): com.moji.moweather, com.moji.moweather.widget.CMojiWidget4x1
Line 1095: 08-23 12:54:39.469 I/test ( 5772): com.moji.moweather, com.moji.moweather.widget.CMojiWidget4x2
```
# 添加widget到某个View中
总共分为5步：
1. 使用AppHost分配appWidgetId。
2. 绑定appWidetId到墨迹天气Intent上。
3. AppWidgetmanaget根据appWidgetId解析出appWidgetInfo。
4. AppWidgetHostView使用appWidgetId和appWidgetInfo创建View。
5. 将上一步创建好的View添加到目标测试View中。
```java
int appWidgetId = Launcher.mAppWidgetHost.allocateAppWidgetId();
Intent intent = new Intent();
intent.setClassName("com.moji.moweather", "com.moji.moweather.widget.CMojiWidget4x2");
Launcher.mAppWidgetManager.bindAppWidgetId(appWidgetId, intent.getComponent());
AppWidgetProviderInfo appWidgetInfo = Launcher.mAppWidgetManager.getAppWidgetInfo(appWidgetId);
Launcher.mAppWidgetHostView = (LauncherAppWidgetHostView) Launcher.mAppWidgetHost.createView(context, appWidgetId, appWidgetInfo);
testView.addView(Launcher.mAppWidgetHostView);// 添加widget到测试view中
```
# 其他：
```java
// 获取系统已安装的AppWidgetManager.ACTION_APPWIDGET_UPDATE的类列表
Intent intent = new Intent(AppWidgetManager.ACTION_APPWIDGET_UPDATE);
String pkgName = "com.moji.moweather";
intent.setPackage(pkgName);
PackageManager pkgManger = context.getPackageManager();
List<ResolveInfo> broadcastReceivers = pkgManger
  .queryBroadcastReceivers(intent, PackageManager.GET_META_DATA);
final int N = broadcastReceivers == null ? 0 : broadcastReceivers
  .size();
for (int i = 0; i < N; i++) {
  ResolveInfo ri = broadcastReceivers.get(i);
  ActivityInfo ai = ri.activityInfo;
  if ((ai.applicationInfo.flags & ApplicationInfo.FLAG_EXTERNAL_STORAGE) != 0) {
  continue;
  }
  Log.i("test", "" + ai.packageName + ", " + ai.applicationInfo.className); //打印包名和类名
  if (pkgName.equals(ai.packageName)) {
  }
}
```
Android原生launcher中的LauncherAppWidgetHost.java和LauncherAppWidgetHostView.java代码如下：
LauncherAppWidgetHost.java
```java
/*
 * Copyright (C) 2009 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
import android.appwidget.AppWidgetHost;
import android.appwidget.AppWidgetHostView;
import android.appwidget.AppWidgetProviderInfo;
import android.content.Context;
/**
 * Specific {@link AppWidgetHost} that creates our {@link LauncherAppWidgetHostView}
 * which correctly captures all long-press events. This ensures that users can
 * always pick up and move widgets.
 */
public class LauncherAppWidgetHost extends AppWidgetHost {
  Launcher mLauncher;
  public LauncherAppWidgetHost(Launcher launcher, int hostId) {
  super(launcher, hostId);
  mLauncher = launcher;
  }
  @Override
  protected AppWidgetHostView onCreateView(Context context, int appWidgetId,
  AppWidgetProviderInfo appWidget) {
  return new LauncherAppWidgetHostView(context);
  }
  @Override
  public void stopListening() {
  super.stopListening();
  clearViews();
  }
  protected void onProvidersChanged() {
  // Once we get the message that widget packages are updated, we need to rebind items
  // in AppsCustomize accordingly.
  mLauncher.bindPackagesUpdated();
  }
}
```
LauncherAppWidgetHostView.java
```java
package com.android.launcher2;
import android.appwidget.AppWidgetHostView;
import android.content.ComponentName;
import android.content.ContentValues;
/**
 * Represents a widget (either instantiated or about to be) in the Launcher.
 */
class LauncherAppWidgetInfo extends ItemInfo {
  /**
  * Indicates that the widget hasn't been instantiated yet.
  */
  static final int NO_ID = -1;
  /**
  * Identifier for this widget when talking with
  * {@link android.appwidget.AppWidgetManager} for updates.
  */
  int appWidgetId = NO_ID;
  ComponentName providerName;
  // TODO: Are these necessary here?
  int minWidth = -1;
  int minHeight = -1;
  private boolean mHasNotifiedInitialWidgetSizeChanged;
  /**
  * View that holds this widget after it's been created. This view isn't created
  * until Launcher knows it's needed.
  */
  AppWidgetHostView hostView = null;
  LauncherAppWidgetInfo(int appWidgetId, ComponentName providerName) {
  itemType = LauncherSettings.Favorites.ITEM_TYPE_APPWIDGET;
  this.appWidgetId = appWidgetId;
  this.providerName = providerName;
  // Since the widget isn't instantiated yet, we don't know these values. Set them to -1
  // to indicate that they should be calculated based on the layout and minWidth/minHeight
  spanX = -1;
  spanY = -1;
  }
  @Override
  void onAddToDatabase(ContentValues values) {
  super.onAddToDatabase(values);
  values.put(LauncherSettings.Favorites.APPWIDGET_ID, appWidgetId);
  }
  /**
  * When we bind the widget, we should notify the widget that the size has changed if we have not
  * done so already (only really for default workspace widgets).
  */
  void onBindAppWidget(Launcher launcher) {
  if (!mHasNotifiedInitialWidgetSizeChanged) {
  notifyWidgetSizeChanged(launcher);
  }
  }
  /**
  * Trigger an update callback to the widget to notify it that its size has changed.
  */
  void notifyWidgetSizeChanged(Launcher launcher) {
  AppWidgetResizeFrame.updateWidgetSizeRanges(hostView, launcher, spanX, spanY);
  mHasNotifiedInitialWidgetSizeChanged = true;
  }
  @Override
  public String toString() {
  return "AppWidget(id=" + Integer.toString(appWidgetId) + ")";
  }
  @Override
  void unbind() {
  super.unbind();
  hostView = null;
  }
}
```