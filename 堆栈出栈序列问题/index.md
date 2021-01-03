# 堆栈出栈序列问题


这是工作中遇到的小问题。

<!--more-->

![](https://github.com/bugxch/blogpics/blob/master/201807/stack.jpg?raw=true)

数据结构中有一种数据类型——**堆栈**，该结构中的数据项有如下特点：

1. 除了最前面和最后面的数据，每个数据项都有一个前驱结点和一个后继结点；

2. 堆栈两端分别称为栈顶和栈底，数据项只能在栈顶加入或者弹出。![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Data_stack.svg/400px-Data_stack.svg.png)

   很明显，堆栈的数据遵循先入后出原则。假设我们有 3 个不同的数据项，编号 1，2，3，只要保证入栈顺序是大编号在后小编号在前，且每次进栈的数量不限，则所有可能的出栈顺序有：1->2->3，1->3->2，2->1->3，2->3->1，3->2->1 一共 5 种，注意 3->1->2 不是可能的出栈顺序，因为如果 3 最先出栈，那么 1 和 2 必在栈中（如果还未入栈，则说明 3 先入栈，与假设矛盾），只能 2 在上 1 在下，所以出栈顺序必然是 2->1。那么，

   > **问题一**：编号$1\sim n$的连续数据项以编号的先后顺序入栈然后出栈，所有可能的出栈顺序有多少种？

   上面的问题比较难于回答，引申之后得到另一个比较弱的问题

   > **问题二**：给定一个长度为$n$ 的整数序列，且各个元素均不相同，它是否是一个出栈序列？

   为了回答以上的两个问题，我们首先来看下一个正常的出栈序列有什么特点。假设长度为 $n$的出栈序列是$a_1,a_2,…,a_n$，取其中第$k$ 个数 $a_k$，则有如下结论：

   1. $a_k$之前的所有数据项都已经出栈，即$a_1,a_2,…,a_{k-1}$都已经出栈；
   2. $a_k$ 之后的所有数据项中，小于 $a_k$的都在栈内，大于$a_k$的尚未入栈；
   3. $a_k$之后紧跟的出栈数据项 $a_{k+1}$ 要么大于$a_k$，要么是所有未出栈的比$a_k$小的数据项中最大的一个

   结论 1 很明显，因为本身就是出栈序列，因此之前的数据肯定已经出栈；结论 2 中，之后的数据只有两种存在的可能：在栈内，或者未进栈。比$a_k$小的如果未进栈，那么 akak 根本不可能出栈（因为就没进栈），比$a_k$大的如果在栈内，那$a_k$也无法出栈，因为$a_k$在它的下面，因此得证；结论 3，$a_{k+1}$就是$a_k$ 出栈后栈顶的数据，因此必然是在栈内的数据的最上面的一个，或者是栈外的某一个数据（进栈再出栈）。

   于是由结论 3 找到判断序列的方法：**逐个检查序列的每一项$a_k$，将该项之后的数据分为大于该数据的大数集合$S_g$和小于该数的小数集合$S_l$，查看是否后续的数据项是小数集合的最大值或者是大数集合的任意值，如果不是则不是出栈序列，即若 $a_{k+1}\in S_g$ 或 $a_{k+1}=max_l{S_l}$，即是出栈序列。**

   问题一的解答，就是穷举长度为 nn 的序列，逐个进行判断，得到最后的结果，附上 python 程序。

   ```python
   import math
   import itertools

   % 输入序列的长度
   n = raw_input("Input n: ")

   % 判断是否是出栈序列
   def IsNotStackSeq(n_ls, n):

       for k in range(0,n-2):
       % 逐个检查序列中的每一个元素
           ak = n_ls[k]
           set_in = n_ls[k+1:]
           a_max = ak

           % 将ak之后的元素分为大于和小于两组集合
           min_list = [item for item in set_in if item > ak]
           max_list = [item for item in set_in if item < ak]

           if len(max_list) > 0:
               a_max = max(max_list)

           % 后续的元素是否是小于集合的最大值或者属于大于集合
           if n_ls[k+1] != a_max and (n_ls[k+1] not in min_list):
               return 1
       return -1

   def StackSeqList(n):
           n_permation = list(itertools.permutations(range(1,int(n)+1), int(n)))
           n_list = [item for item in n_permation if IsNotStackSeq(list(item),int(n)) < 0]
           return (len(n_list),n_list)
   ```

