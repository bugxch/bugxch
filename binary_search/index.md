# 二分搜索——从入门到精通

最近在准备考试，做了一些二分搜索的题目，感觉有点意思，记录下做题的心得。

<!--more-->
## 文章目标

通过阅读本文，可以彻底搞懂二分查找的基本原理及各种变体，可以独立完成下面的力扣题目

| 力扣题目                                                     | 考查点                                 |
| ------------------------------------------------------------ | -------------------------------------- |
| [374. 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/) | 基本模板                               |
| [69. x 的平方根](https://leetcode-cn.com/problems/sqrtx/)    | 基本模板                               |
| [278. 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china) | 查找左边界                             |
| [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/) | 寻找旋转数组中的目标值（无重复元素）   |
| [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/) | 寻找旋转数组中的目标值（包含重复元素） |
| [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | 查找左右边界                           |

## 基本原理
在计算机科学中，二分搜索又被成为**半区间**搜索，对数搜索或者二分chop，它用于在排序数列中找到目标值的位置。算法不断比较数列中间元素与目标值，

1. 如果目标值与中间元素匹配，那直接返回中间的位置；
2. 如果目标值比中间元素小，继续搜索低半边的数列；
3. 如果目标值比中间元素大，那么继续搜索高半边

注意到**每次在比较元素之后搜索的区间会减少一半（去掉目标值不可能在的那一半区间）**，所以在最坏的情况下，算法的复杂度是$O(logn)$。下面是二分搜索与线性搜索的比较示意图，在同一个排序的数列上，查找目标值为37的序列值。

![](https://devopedia.org/images/article/28/2951.1490520804.gif)

序列长度是17，
1. 二分查找第一次寻找整个搜索区间的中间的index（从0开始计数）为(0 + 16) / 2 = 8的数字23，23比37小，所以**更新查找区间为**[9,16]；
2. 查找新区间的中间index = (9 + 16) /2 = 12，结果是数字41，比37大，**更新查找区间**为[9, 11];
3. 查找新区间的中间index = (9 + 11) / 2 = 10，结果是数字31，比37大，继续**更新查找区间**为[11, 11]，此时中间元素就是第11号元素，即37，找到目标值，查找结束。

根据以上算法步骤，很顺利就可以写出下面的伪代码，

```shell
// basic template
function binary_search(A, n, T) is
    L := 0
    R := n − 1
    while L ≤ R do
        m := floor((L + R) / 2)
        if A[m] < T then
            L := m + 1
        else if A[m] > T then
            R := m − 1
        else:
            return m
    return unsuccessful
```

使用上面的基本模板我们就可以解答文章开始列出的第一道题目了，[374. 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)题目，给出二分查找的基本模板。这道题目就是最传统的猜大小的谜题，标准的解答模板的代码如下
```cpp
/**
 * Forward declaration of guess API.
 * @param  num   your guess
 * @return 	     -1 if num is lower than the guess number
 *			      1 if num is higher than the guess number
 *               otherwise return 0
 * int guess(int num);
 */

class Solution {
public:
    int guessNumber(int n) {
        int left = 1;
        int right = n;

        while (left <= right) { // detail 1
            int mid = left + ((right - left) >> 1); // detail 2
            int rlt = guess(mid);
		
            if (rlt == 0) {
                return mid;
            }
			// detail 3
            if (rlt == -1) {
                right = mid - 1;
            }  else {
                left = mid + 1;
            }
        }
        return -1;
    }
};
```

### 关键技术细节

> *Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky*  -- **Donald Knuth**

就像高德纳所说，二分法的思想简单且易于理解，但是二分法的细节却藏了很多坑，**魔鬼就在细节中**。为了透彻理解二分法的实现，有必要对上面的代码的下面3个技术细节（标记为`detail`注释的地方）进行深究。
#### 循环的入口条件
代码16行，为什么循环的入口条件是`left <= right`？

>  无论何时$[left, right]$表示**可能包含目标值**的搜索区间

注意这是个**左闭右闭**的闭区间，单纯从数学角度出发，这个区间**最短的长度就是1**，也就是$left = right$的时候。如果$left >  right$了，那$[left, right]$就是**空集**，这个集合**肯定不包含目标值**，也就**没有继续搜索的必要**了，所以循环退出。

#### 中点位置
为什么是`mid = left + ((right - left) >> 1)`，也就是数学的$floor((left + right) / 2)$?
这里的中点转换成数学表示就是
$$
\begin{align}
mid = \left \lfloor\frac{left + right}{2}  \right \rfloor 
\end{align}
$$
其实，我们还有另外一种选择，将中间值定为
$$
\begin{align}
mid = \left \lceil\frac{left + right}{2}  \right \rceil 
\end{align}
$$
第2种方式的中点值的选择是否可行？从后面的分析，**其实是可行的。**那这两个计算公式有什么区别？最大的区别在于$right = left + 1$的时候，也就是搜索区间长度为2左右端点挨着的时候，如下图所示，下一次循环计算mid，第1个公式结果是$mid = 4$，而第2个公式的结果是$mid = 5$，

![指针相邻](https://pic.imgdb.cn/item/609680d8d1a9ae528f734868.png)

，第1个公式$mid = \left \lfloor\frac{left + right}{2}  \right \rfloor = \left \lfloor\frac{left + left + 1}{2}  \right \rfloor = left$，而第2个公式$mid = \left \lceil\frac{left + right}{2}  \right \rceil = \left \lceil\frac{left + left + 1}{2}  \right \rceil = left + \lceil 0.5\rceil = left + 1 = right$，请记住这个重要的结论。

> 当搜索区间的**左右两端点挨着**的时候（即$right = left + 1)$，floor函数计算的中点mid是left，ceil函数计算的中点mid是right

请记住这个**特殊的区间情况**，至于为何基础模板选择floor而没有选择ceil，在后面的变化类型一节很关键。

#### 搜索区间调整
在每次判断与target的差距之后，搜索区间为什么是这么调整的？
二分法另一个容易搞错的问题是区间调整，$left$和$right$好像可以选择$mid$的3个边界值$mid, mid - 1, mid + 1$中的任意一个，其实**调整的原则**很简单，

> 1. **排除**target不可能存在的区间，**保留**可能存在的区间
> 2. 结合中点函数的计算原则，需要保证**任意判断分支为true时最后都可以跳出循环，尤其当$right = left + 1$时**

**第2条简直就是区间调整和中点选择的试金石**。基础模板中，根据mid的值与目标值的差距，搜索区间$[left, right]$的调整有3个判断分支：

1. 如果和目标值相同$A[mid] = A[target]$，直接**跳出循环**，返回mid(代码第21行)；
2. 如果比目标值大$A[mid] > A[target]$，更新$right = mid - 1$，搜索区间长度进一步减少；
3. 如果比目标值小$A[mid] < A[target]$，更新$left = mid + 1$，搜索区间长度进一步减少

分支3的调整情况，看如下图示

![](https://pic.imgdb.cn/item/609686fad1a9ae528fbc00c9.png)

因为我们查找的是一个严格单增的序列，所以如果mid的值比target小，那么可以肯定

1. 很显然，target必然不在左半边$[left, mid - 1]$上，所以**排除这个不可能的区间**，包含target的区间变成$[mid, right]$；
2. 新搜索区间不包含mid，因为$A[mid] < A[target]$，而且是严格单增函数，所以$mid < target$，所以$mid$必然不是$target$，区间进一步缩小为$[mid + 1, right]$。

分支2情况的调整策略与分支3的同理，此处不再赘述。需要注意，这样的策略**是否可以退出循环**？这个循环的退出条件是$left > right$。对照上面的3个判断分支分别为true的情况：

1. 第1个判断分支直接退出循环，符合条件；
2. 同理，第2个判断分支每次判断之后$right$在变小，所以必然在若干次之后比$left$小，直至退出循环
3. 第3个判断分支每次调整之后$left$在变大，所以所以必然在若干次之后比$right$大，直至退出循环

所以随着区间的调整，必然会在某一步满足退出的条件。**特别的**，当$right = left + 1$时，$mid = left$；

- 第2个判断分支，$right = mid - 1 = left - 1 = right - 2 < left = right - 1$，跳出循环
- 第3个判断分支，$left = mid + 1 = left + 1 = right$，再次进入循环，此时$mid = left = right$，$left = mid + 1 = right + 1 > right$，可以看到在迭代2次之后也退出循环

满足上面的条件2。

---

结合上面的分析，按照如下的步骤写出二分查找，判断$A[mid]$与$A[target]$大小的3种情况，确定每个分支的区间调整策略，结合中点的计算规则，满足如下2个条件，

> 1. **排除**target不可能存在的区间，**保留**可能存在的区间
> 2. 结合中点函数的计算原则，需要保证**任意判断分支为true时最后都可以跳出循环，尤其当$right = left + 1$时**

### 深入探究

#### 分支合并

当前的基础代码有3个判断分支，是否可以将3个分支合并成2个呢？下面的基础模板就可以。

```cpp
class Solution {
public:
    int guessNumber(int n) {
        int left = 1;
        int right = n;

        while (left < right) { // 退出条件变了
            int mid = left + ((right - left) >> 1);
            // 调整策略的分支从3个变成了2个
            if (guess(mid) <= 0) {
                right = mid;
            }
            else {
                left = mid + 1;
            }
        }
		// 返回的条件也变了
        if (guess(left) == 0) {
            return left;
        }

        return -1;
    }
};
```

再仔细考察一下，上面的代码，3个地方发生变化（已经在注释种标出）依然可以通过测试，其实还是按照上面的3个原则来看。

1. 每次调整区间，**排除**target不可能存在的区间，**保留**可能存在的区间

   重新分析之前的3个判断分支，比较特殊是第1点，如果$A[mid] = A[target]$，那么$mid = target$，很明显

   - 判断分支1与判断分支2合并，当$A[mid] \ge A[target]$，$target \le mid$，所以$target$**可能位于**$[left, mid]$区间，调整$right = mid$，注意此处$right \ne mid -1$
   - 判断分支1与判断分支3合并，当$A[mid] \le A[target]$，$target \ge mid$，所以$target$**可能位于**$[mid, right]$区间，调整$left = mid$，注意此处$left \ne mid +1$

   这两个策略看起来似乎都可以，结合其他两种情况，如果将3个分支合并为2个，应该有2种调整策略

   ```shell
   # solution 1，另一种正确的方案
   function binary_search_alternative(A, n, T) is
       L := 0
       R := n − 1
       while L < R do
           m := floor((L + R) / 2)
           if A[m] >= T then
               R := m
           else:
               L := m + 1
       if A[L] = T then
           return L
       return unsuccessful
   ```

   或者

   ```shell
   # solution 2，错误示范
   function binary_search_alternative(A, n, T) is
       L := 0
       R := n − 1
       while L < R do
           m := floor((L + R) / 2)
           if A[m] <= T then
               L := m
           else:
               R := m - 1
       if A[L] = T then
           return L
       return unsuccessful
   ```

2. 结合中点函数的计算原则，需要保证**任意判断分支为true时最后都可以跳出循环，尤其当$right = left + 1$时**

   我们再看上面的两种伪代码，退出的条件是$left >= right$，直接考察$left = right -1$的情况，如果在搜索的若干步之后搜索的区间变成下面的情况

   ![指针相邻](https://pic.imgdb.cn/item/609680d8d1a9ae528f734868.png)

   下一步搜索的$mid = 4$，按照方案2，如果此时

   1. 第2个判断分支为true，$right = mid - 1 = 3 < left$，退出循环；
   2. 第1个判断分支为true（代码第7~8行），那么调整$left = mid = 4$，再次进入之后**会发现陷入死循环**，核心原因在于中点的选择上，因为使用的是floor函数，导致left永远恒等于mid，换句或说，$left$永远不再增加，而且也进不到更新$right$的分支，陷入死循环，考察方案1就没有这种情况。那如果是这样，我们改造下方案2，将中点的函数选成ceil是不是就ok了？
   
   ```shell
   # solution 3，solution 2的改造
   function binary_search_alternative(A, n, T) is
       L := 0
       R := n − 1
       while L < R do
           m := ceil((L + R) / 2) # ceil取代floor函数
           if A[m] <= T then
               L := m
           else:
               R := m - 1
       if A[L] = T then
           return L
       return unsuccessful
   ```
   
   再考察一下上面的搜索区间左右端点相邻的情况，下一次搜索的$mid = 5$，无论走哪一个判断分支，最终$left = right$，跳出循环，所以
   
   > 中点的选择需要结合区间调整的策略来看，保证无论哪个分支为true**都不能出现死循环**

#### 判断条件变更

需要注意，其他2个的模板的循环条件为$left < right$，当前$left <= right$时退出，如果$left = right$，那么搜索的区间只剩下一个元素，说明其他的元素不符合条件。如果最后的这个元素依然不是target，那么所有的元素也不是了,所以在最后我们加了一个判断，

```shell
    if A[L] = T then
        return L
    return unsuccessful
```

## 变种题目

理解上面的分析过程之后，进入进阶版的二分法题目。

### 重复元素左边界

参考[278. 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china)，从某个版本开始，版本就已经不可用了，但是在这个版本之前，所有的版本均可用，需要找出第一个不可用的版本。假如说，我们有100个版本，从第70个版本开始不用，那么怎么快速找到70呢？如下所示

![](https://pic.imgdb.cn/item/609726b8d1a9ae528f1f0aa7.png)

题目要找到最左边第一个true对应的下标70。

### 重复元素右边界

### 旋转数组（无重复元素）

### 旋转数组（ 包含重复元素）左边界

## 参考资料

1. [二分查找、二分边界查找算法的模板代码总结 - SegmentFault 思否](https://segmentfault.com/a/1190000016825704)，给我很多启发的一篇文章
2. [Binary search algorithm](https://en.wikipedia.org/wiki/Binary_search_algorithm)，维基百科页面，英文版里面的内容很详尽
---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
