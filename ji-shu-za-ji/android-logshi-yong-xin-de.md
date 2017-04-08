2015-10-30 Android Log使用心得

# android 如何打印堆栈信息
通过如下方法，可以直接打印出程序运行到当前点的堆栈信息，方便调试：
```java
RuntimeException here = new RuntimeException("here");
here.fillInStackTrace();
Log.i(TAG, "test", here);
```
# Log扩展类，自动打印类名，方法名称
```
import android.util.Log;
/**
 * Log打印工具，可以快速打印log及其所在的方法名称。
 * 调用示例LogExt.LogD(this, Thread.currentThread().getStackTrace(), ...);
 * @author yyxz
 */
public class LogExt {
  private static final String TAG= "yyxz";
  private static boolean CanShowLog(){return isShowLog;};
  private static boolean isShowLog = true;
  // Log Debug
  public static void LogD(Object obj, StackTraceElement[] sTE) {
  LogD(obj, sTE, null);
  }
  public static void LogD(Object obj, StackTraceElement[] sTE, String strInfo) {
  LogD(obj, sTE, strInfo, null);
  }
  public static void LogD(Object obj, StackTraceElement[] sTE, String strInfo, Throwable tr) {
  if(!CanShowLog()) {
  return ;
  }
  Class cls;
  if (obj instanceof Class) {
  cls = ( Class)obj;
  } else {
  cls = obj.getClass();
  }
  String methodName = sTE[2].getMethodName();
  if ((cls == null) || (methodName == null)) {
  Log. e(TAG, "Input Class or MethodName Error(null)" );
  } else {
  if (strInfo == null) {
  if (tr == null) {
  Log. d(TAG, cls.toString() + "::" + methodName);
  } else {
  Log. d(TAG, cls.toString() + "::" + methodName, tr);
  }
  } else {
  if (tr == null) {
  Log. d(TAG, cls.toString() + "::" + methodName + "(), " + strInfo);
  } else {
  Log. d(TAG, cls.toString() + "::" + methodName + "(), " + strInfo, tr);
  }
  }
  }
  }
  // Log Error
  public static void LogE(Object obj, StackTraceElement[] sTE) {
  LogE(obj, sTE, null);
  }
  public static void LogE(Object obj, StackTraceElement[] sTE, String strInfo) {
  LogE(obj, sTE, strInfo, null);
  }
  public static void LogE(Object obj, StackTraceElement[] sTE, String strInfo, Throwable tr) {
  if(!CanShowLog()) {
  return ;
  }
  Class cls;
  if (obj instanceof Class) {
  cls = ( Class)obj;
  } else {
  cls = obj.getClass();
  }
  String methodName = sTE[2].getMethodName();
  if ((cls == null) || (methodName == null)) {
  Log. e(TAG, "Input Class or MethodName Error(null)" );
  } else {
  if (strInfo == null) {
  if (tr == null) {
  Log. e(TAG, cls.toString() + "::" + methodName);
  } else {
  Log. e(TAG, cls.toString() + "::" + methodName, tr);
  }
  } else {
  if (tr == null) {
  Log. e(TAG, cls.toString() + "::" + methodName + "(), " + strInfo);
  } else {
  Log. e(TAG, cls.toString() + "::" + methodName + "(), " + strInfo, tr);
  }
  }
  }
  }
  // Log Info
  public static void LogI(Object obj, StackTraceElement[] sTE) {
  LogI(obj, sTE, null);
  }
  public static void LogI(Object obj, StackTraceElement[] sTE, String strInfo) {
  LogI(obj, sTE, strInfo, null);
  }
  public static void LogI(Object obj, StackTraceElement[] sTE, String strInfo, Throwable tr) {
  if(!CanShowLog()) {
  return ;
  }
  Class cls;
  if (obj instanceof Class) {
  cls = ( Class)obj;
  } else {
  cls = obj.getClass();
  }
  String methodName = sTE[2].getMethodName();
  if ((cls == null) || (methodName == null)) {
  Log. e(TAG, "Input Class or MethodName Error(null)" );
  } else {
  if (strInfo == null) {
  if (tr == null) {
  Log. i(TAG, cls.toString() + "::" + methodName);
  } else {
  Log. i(TAG, cls.toString() + "::" + methodName, tr);
  }
  } else {
  if (tr == null) {
  Log. i(TAG, cls.toString() + "::" + methodName + "(), " + strInfo);
  } else {
  Log. i(TAG, cls.toString() + "::" + methodName + "(), " + strInfo, tr);
  }
  }
  }
  }
  // Log Version
  public static void LogV(Object obj, StackTraceElement[] sTE) {
  LogV(obj, sTE, null);
  }
  public static void LogV(Object obj, StackTraceElement[] sTE, String strInfo) {
  LogV(obj, sTE, strInfo, null);
  }
  public static void LogV(Object obj, StackTraceElement[] sTE, String strInfo, Throwable tr) {
  if(!CanShowLog()) {
  return ;
  }
  Class cls;
  if (obj instanceof Class) {
  cls = ( Class)obj;
  } else {
  cls = obj.getClass();
  }
  String methodName = sTE[2].getMethodName();
  if ((cls == null) || (methodName == null)) {
  Log. v(TAG, "Input Class or MethodName Error(null)" );
  } else {
  if (strInfo == null) {
  if (tr == null) {
  Log. v(TAG, cls.toString() + "::" + methodName);
  } else {
  Log. v(TAG, cls.toString() + "::" + methodName, tr);
  }
  } else {
  if (tr == null) {
  Log. v(TAG, cls.toString() + "::" + methodName + "(), " + strInfo);
  } else {
  Log. v(TAG, cls.toString() + "::" + methodName + "(), " + strInfo, tr);
  }
  }
  }
  }
  // Log Warn
  public static void LogW(Object obj, StackTraceElement[] sTE) {
  LogW(obj, sTE, null);
  }
  public static void LogW(Object obj, StackTraceElement[] sTE, String strInfo) {
  LogW(obj, sTE, strInfo, null);
  }
  public static void LogW(Object obj, StackTraceElement[] sTE, String strInfo, Throwable tr) {
  if(!CanShowLog()) {
  return ;
  }
  Class cls;
  if (obj instanceof Class) {
  cls = ( Class)obj;
  } else {
  cls = obj.getClass();
  }
  String methodName = sTE[2].getMethodName();
  if ((cls == null) || (methodName == null)) {
  Log. e(TAG, "Input Class or MethodName Error(null)" );
  } else {
  if (strInfo == null) {
  if (tr == null) {
  Log. w(TAG, cls.toString() + "::" + methodName);
  } else {
  Log. w(TAG, cls.toString() + "::" + methodName, tr);
  }
  } else {
  if (tr == null) {
  Log. w(TAG, cls.toString() + "::" + methodName + "(), " + strInfo);
  } else {
  Log. w(TAG, cls.toString() + "::" + methodName + "(), " + strInfo, tr);
  }
  }
  }
  }
}
```