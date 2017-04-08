2015-12-3 获取Android已安装的app信息

# 获取包含activity应用的信息
```java
PackageManager pm = getPackageManager(); // 获得PackageManager对象
Intent mainIntent = new Intent(Intent. ACTION_MAIN, null );
mainIntent.addCategory(Intent.CATEGORY_LAUNCHER);
List<ResolveInfo> resolveInfos = pm
  .queryIntentActivities(mainIntent, 0);
appList.clear();
for (ResolveInfo reInfo : resolveInfos) {
  // 可能一个包有多个图标
  LogExt. LogD(this, Thread. currentThread().getStackTrace(),
  "reInfo.activityInfo.name=" + reInfo.activityInfo.name );
  String activityName = reInfo. activityInfo.name ; // 获得该应用程序的启动Activity的name
  String appLabel = (String) reInfo.loadLabel(pm); // 获得应用程序的Label
  Drawable icon = reInfo.loadIcon(pm); // 获得应用程序图标
  // 创建一个AppInfo对象，并赋值
  AppInfo tmpInfo = new AppInfo();
  tmpInfo. appName = appLabel;
  tmpInfo. packageName = reInfo.activityInfo .packageName ;
  tmpInfo. mainActivityName = activityName;
  tmpInfo. appIcon = icon;
  appList.add(tmpInfo);
}
```
# 获取所有的App信息
```java
PackageManager pm = getPackageManager();
List<PackageInfo> packages = pm
// .getInstalledPackages(0);
  .getInstalledPackages(PackageManager. MATCH_DEFAULT_ONLY);
LogExt.LogD (this , Thread.currentThread ().getStackTrace(),
  "packages.size() = " + packages.size());
appList.clear();
for (int i = 0; i < packages.size(); i++) {
  PackageInfo packageInfo = packages.get(i);
  AppInfo tmpInfo = new AppInfo();
  tmpInfo. appName = packageInfo.applicationInfo .loadLabel(
  getPackageManager()).toString();
  tmpInfo. packageName = packageInfo.packageName ;
  tmpInfo. versionName = packageInfo.versionName ;
  tmpInfo. versionCode = packageInfo.versionCode ;
  tmpInfo. appIcon = packageInfo.applicationInfo
  .loadIcon(getPackageManager());
  appList.add(tmpInfo);
}
```