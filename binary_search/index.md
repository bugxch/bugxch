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
在计算机科学中，二分搜索又被成为**半区间**搜索，对数搜索或者二分chop，它用于在排序数列中找到目标值的位置。基本的算法包含如下步骤：二分搜索比较数列中间元素与目标值，如果目标值与中间元素匹配，那直接返回中间的位置。如果目标值比中间元素小，继续搜索低半边的数列，如果目标值比中间元素大，那么继续搜索高半边。**每次在比较元素之后搜索的区间会减少一半（去掉目标值不可能在的那一半区间）**，所以在最坏的情况下，算法的复杂度是$O(logn)$。下面是二分搜索与线性搜索的比较示意图，在同一个排序的数列上，查找目标值为37的序列值。

![](https://devopedia.org/images/article/28/2951.1490520804.gif)

序列长度是17，
1. 二分查找第一次寻找整个搜索区间的中间的index（从0开始计数）为(0 + 16) / 2 = 8的数字23，23比37小，所以**更新查找区间**[9,16]；
2. 查找新区间的中间index = (9 + 16) /2 = 12，结果是数字41，比37大，**更新查找区间**
[9, 11];
3. 查找新区间的中间index = (9 + 11) / 2 = 10，结果是数字31，比37大，继续**更新查找区间**[11, 11]，此时中间元素就是第11号元素，即37，找到目标值，查找结束。

参考上面的算法步骤，可以看出二分搜索的关键步骤包括**比较目标值**以及**更新区间**两部分，原理在下面的图片中给出，

![基本模板](https://pic.imgdb.cn/item/6095cad8d1a9ae528fbd7568.png)

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

        while (left <= right) {
            int mid = left + ((right - left) >> 1);
            int rlt = guess(mid);

            if (rlt == 0) {
                return mid;
            }

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

### 区间更新谜题

> *Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky*  -- **Donald Knuth**

二分法很容易在左右区间的选择上出现问题，如果想真正理解二分查找，那么需要理解下面的**关键细节**:
> 1. 如何保证在任意情况下，`while`的判断条件不会陷入死循环？
> 2. 在每次判断与target的差异之后，如何变更区间更高效？

### 变化类型



## 实践题目
## 参考资料

1. [二分查找、二分边界查找算法的模板代码总结 - SegmentFault 思否](https://segmentfault.com/a/1190000016825704)，给我很多启发的一篇文章
2. [Binary search algorithm](https://en.wikipedia.org/wiki/Binary_search_algorithm)，维基百科页面，英文版里面的内容很详尽
---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
