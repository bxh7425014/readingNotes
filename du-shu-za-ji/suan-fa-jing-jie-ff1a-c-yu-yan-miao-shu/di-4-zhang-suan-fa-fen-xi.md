第4章 算法分析
# 最坏情况分析
最坏情况告诉我们算法的性能上限。

# O表示法的简单规则
- 常数项O(1)
$$
O(c) = O(1)
$$
    
    一个算法无论处理多大的数据量，都消耗固定时间。
    
- 忽略常量因子
$$
O(cT) = cO(T) = O(T)
$$

    执行相同次数的有些任务。
    
- 加法运算取最大值
$$
O(T_1)+O(T_2) = O(T_1+T_2)= max(O(T_1),O(T_2))
$$

    两个顺序执行的任务。
- 乘法结果
$$
O(T_1)O(T_2)=O(T_1T_2)
$$
    
    比如一个嵌套循环。

# 计算的复杂度
O表示法能够描述一个算法的复杂度。谈论算法的性能时，我们关注的也是它。复杂度与它所处理的数据量所需要的资源（通常是时间）的增长速率密切相关。
![](/assets/QQ20170519-161945.jpg)
![](/assets/QQ20170519-162044.jpg)
![](/assets/QQ20170519-162101.jpg)
在给定一个标准的情况下，能够使此算法表现最佳时，我们就认为此算法是高效的。一个高效的实现并不总能影响算法的复杂度，但它可以降低常量因素的影响，从而使算法在实际应用中更加高效。

# 相关主题
- NP完全问题

    没有已知的求解多项式时间的算法，但也无法证明此多项式不存在，这类问题称为NP完全问题。