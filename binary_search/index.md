# 二分法学习心得

最近在准备考试，做了一些二分搜索的题目，感觉有点意思，记录下做题的心得。
<!--more-->
二分法很容易在左右区间的选择上出现问题，我在做完一些题目之后，发现针对不同的情况需要使用不同的区间的处理策略，在这里记录下。

## 基本原理
二分法适用于在单调递增或者递减的序列以$O(logn)$的复杂度快速找到目标值。注意，这里的使用情景包括了严格或者非严格的情况，我们举例说明。

我们通过解答力扣上的这道[374. 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)题目，给出二分查找的基本模板。这道题目就是最传统的猜大小的谜题，标准的解答模板的代码如下
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
### 理解要点

如果想真正理解二分查找，那么需要真正理解下面的**关键细节**:
> 1. 如何保证在任意情况下，`while`的判断条件不会陷入死循环？
> 2. 在每次判断与target的差异之后，如何变更区间更高效？

### 变化类型



## 实践题目
## 参考资料

1. [二分查找、二分边界查找算法的模板代码总结 - SegmentFault 思否](https://segmentfault.com/a/1190000016825704)，给我很多启发的一篇文章
2. [Binary search algorithm](https://en.wikipedia.org/wiki/Binary_search_algorithm)，维基百科页面，英文版里面的内容很详尽
---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
