2015-10-28 使用Excel来计算加班时间，剥离午休时间段.md

# 需求
由于行政mm在整理同事们加班时间Excel表格的时候，不想要统计12:00~13:00时段。产生需求。
因此，晚上加班研究了一下此问题。百度得知Excel有接收正则表达式的语法。

# 实施
## 绘制草图
把可能性都罗列了一遍，绘制草图如下：
![这里写图片描述](http://img.blog.csdn.net/20151028210409510)
Excel只接受类似于正则表达式的方法编写公式，写法类似如下：
- 上班的考勤结果计算公式：
```
=IF(AND(D2>=VALUE("07:35"))=TRUE,"迟到",IF(D2=0,"未打卡",""))
```
> 此公式的意思是，如果“D2”格，即上班时间列中的时间大于等于“7：35”，则显示“迟到”，如果“D2”格中无数据，即为“0”的时候，则显示“未打卡”，以上两个条件都不符合的时候，则显示为空白，即正常上班的意思；
- 下班的考勤结果计算公式：
```
=IF(AND(E2>=VALUE("16:30"))=TRUE,"加班",IF(E2=0,"未打卡","早退"))
```
> 此公式的意思是，如果“E2”格，即下班时间列中的时间大于等于“16：30”，则显示为“加班”，如果“E2”格中无数据，即为“0”的时候，则显示“未打卡”，以上两个条件都不符合的时候，则显示为“早退”

## 代码模拟
根据草图来看，由于Excel只接受if，不接受else if 写法，所以用Eclipse进行代码模拟，编写代码如下：
```
  /**
  * @param B2 代表B2单元格
  * @param C2 代表C2单元格
  * @return 处理结果
  */
  public int test(int B2, int C2) {
  int R = 0;
  if (C2<=B2) { //结束时间早于起始时间的话，结果为0
  R = 0;
  } else {
  if (B2<=12) { // 12表示12点整
  if (C2<=12) {
  R=C2-B2;
  } else {
  if(C2>13) { //13表示13点整
  R = C2-B2-1; //1表示1个小时
  } else {
  R=12-B2;
  }
  }
  } else {
  if (B2>13) {
  R=C2-B2;
  } else {
  if (C2>13) {
  R=C2-13;
  } else {
  R=0;
  }
  }
  }
  }
  return R;
  }
```

## 代码转换
将上面编写的代码转换浓缩为Excel能够接受的公式，转换结果如下：
```
D2=IF(C2<=B2,0,IF(B2<=TIME(12,0,0),IF(C2<=TIME(12,0,0),C2-B2,IF(C2>TIME(13,0,0),C2-B2-TIME(1,0,0),TIME(12,0,0)-B2)),IF(B2>TIME(13,0,0),C2-B2,IF(C2>TIME(13,0,0),C2-TIME(13,0,0),0))))
```

# 输出

## 编写时间进行测试
结果截图如下：
![这里写图片描述](http://img.blog.csdn.net/20151028210432766)

## Excel示例文档
[加班统计表格测试.xlsx](http://yunpan.cn/cFbKSEQSV5Ln3)
http://yunpan.cn/cFbKSEQSV5Ln3 访问密码 2444