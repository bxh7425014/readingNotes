第3章 递归

#基本递归
函数调用自身。
$$
n!=(n)(n-1)(n-2)···(1)
$$
阶乘用递归形式可以定义为：
$$
F(n) = 
\left\{
\begin{array}{a}
1\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 如果n=0,n=1\\
nF(n-1) \ 如果n>1
\end{array}
\right.
$$
递归过程中的两个的基本阶段：递推和回归。
在递推阶段，每个递推调用通过进一步调用自己来记住这次递推过程，当其中有调用满足基本条件时，递推结束。一旦递推阶段结束，处理过程就进入回归阶段，在这之前的函数以逆序的方式回归，直到最初调用的函数返回为止，此时递归过程结束。

![递推与回归](https://git.oschina.net/uploads/images/2017/0419/212943_dd0cf9e3_438941.png "递推与回归")

C程序在内存中，由四个区域组成：代码段、静态数据区、堆与栈。

栈上的那块存储空间成为活跃记录（栈帧），由五个区域组成：输入参数、返回值空间、计算表达式时用到的临时存储空间、函数调用时保存的状态信息以及输出参数。函数调用时产生的活跃记录将一直存于栈中中直到这个函数调用结束。

![输入图片说明](https://git.oschina.net/uploads/images/2017/0419/214645_46e04ab5_438941.png "在这里输入图片标题")

![输入图片说明](https://git.oschina.net/uploads/images/2017/0419/215011_6130ca3c_438941.png "在这里输入图片标题")

栈维护了每个函数调用的信息，直到函数返回后才释放，这需要占有相当大的空间。

#尾递归
一个函数中所有递归形式的调用都出现在函数的末尾。大多数编译器能够识别尾递归，从而为其产生优化的代码。当编译器检测到尾递归的时候，它就覆盖当前的活跃记录而不是在栈中去创建一个新的。

使用尾递归来计算阶乘：
$$
F(n,a) = 
\left\{
\begin{array}{a}
a\ \ \ \ \ \ \ \ \ \ \ 如果n=0,n=1\\
nF(n-1,na) \ 如果n>1
\end{array}
\right.
$$

![输入图片说明](https://git.oschina.net/uploads/images/2017/0419/221235_5890f75d_438941.png "在这里输入图片标题")

![输入图片说明](https://git.oschina.net/uploads/images/2017/0419/221126_691cfdd6_438941.png "在这里输入图片标题")



