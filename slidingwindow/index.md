# 滑动窗口技巧


今天主要刷leetcode的滑动窗口的题目，需要完成如下题目，本文主要记录解题思路和方法，以便加深理解记忆，答案到处都是，只有消化了才是自己的。

<!--more-->

![](https://www.cxyxiaowu.com/wp-content/uploads/2020/04/1587885580-7ad02b4170d7e1a.jpeg)

## 引子

首先来看leetcode中No. 3的**无重复字符的最长子串**这个题目，参考[题目链接](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)，题目很简单

> 给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

```shell
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

拿到题目的第一反应就是暴力解法，

1. 先暴力穷举所有的子串$S_i = [S[left], S[right])$；   

2. 判断这个子串，如果不含重复字符就记录子串长度$l_i$，此处判断是否有重复字串的时间复杂度是$O(n)$；

3. 找出所有记录的子串长度的最大值$max_i\{l_i\}$。
    一段伪代码如下，

  ```c++
 int maxlen = -1;
  for (int left = 0; left < S.length(); left++)
      for (int right = left + 1; right < S.length(); right++) {
       if(window[left, right) contain 重复字符) continue;
       if(right - left > maxlen) {
           maxlen = right -left;
       }
      }
  return maxlen;
  ```

  很明显，该方法的时间复杂度是$O(n^3)$，非常费时。这个算法的复杂度主要来自于第一步，仔细想想那个子串伪代码中的第3行有点问题，伪代码可以看作我们用一个窗口截取所有子串，left表示窗口的左边界（包含），right表示窗口的右边界（不包含），如果窗口中含有重复子串，那么不应该继续扩展右边界的，所以right不能一直向右**扩展**，此时应该向右移动左边界，**缩小**窗口的长度。以示例中给出的字符串为例，参考下面的示意图

![](https://raw.githubusercontent.com/bugxch/blogpics/master/202008/%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE_%E9%80%89%E6%8B%A9%E5%8C%BA%E5%9F%9F_20200802171014.png)

  所以我们有一个更通用的实现框架。

## 通用框架

维护一个**滑动窗口**，

 >  1. 窗口的边界是$[left, right)$，刚开始窗口的长度为0，即left = right = 0；
 >  2. 维护一个哈希表table，用于记录窗口中的字符的统计情况，比如上面的图一中就`table['a'] = 2, table['b'] = 1, table['c'] = 1`；
 > 3. 窗口可**扩展**可**收缩**，
 >   - 如果当前窗口中没有重复字符，则窗口扩展，`right++`，更新哈希表；
 >   - 如果当前窗口中有重复字符，那么窗口收缩，`left++`，更新哈希表；
 > 4. 直到`right`超过需要遍历的字符串的边界为止。

 需要注意，上面的扩展和收缩的时机，对于当前的这个程序，如何判断当前的窗口中是否有重复字符呢？一般的想法是遍历每个键的值，如果有大于1的值就认为有重复字符，而且是在窗口位置发生变化的时候触发遍历动作。其实，这里有一个小技巧，**键值只有在向右扩展的时候才会增长**，刚开始窗口是没有键的，或者即便有也只能是1，所以在窗口扩展的时候，只要去查看新加入的`right`位置的字符的键值是否超过1即可。通过以上分析，我们有了下面的新的伪代码

```C++
int left = 0, right = 0;
int maxlen = -1;

while (right < s.size()) {
    // 增大窗口
    window.add(s[right]);
    right++;
    if(window doesnot contain repeat charactor) {
        maxlen = max(maxlen, right - left);
    }

    while (window needs shrink) {
        // 缩小窗口
        window.remove(s[left]);
        left++;
    }
}

return maxlen;
```

将上面的伪代码翻译成最后的c++代码，列示如下

```c++
int lengthOfLongestSubstring(string s)
{
    int maxlen = 0;
    unordered_map<char, int> window;
    int right = 0, left = 0;
    while (right < s.length())
    {
        auto rch = s[right];
        right++;
        window[rch]++;
        
        if (window[rch] <= 1)
        {
            maxlen = max(maxlen, right - left);
        } else {
            while (window[rch] > 1)
            {
                auto lch = s[left];
                left++;
                window[lch]--;
            }
        }
    }
    return maxlen;
}
```

## 类似题目

下面使用上面的框架解答一下其他的滑动窗口的类似题目

### 最小覆盖子串 - 力扣（LeetCode）

题目见[76. 最小覆盖子串 - 力扣（LeetCode）](https://leetcode-cn.com/problems/minimum-window-substring/)：给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字符的最小子串。

示例：

> **输入:** S = "ADOBECODEBANC", T = "ABC"
> **输出:** "BANC"

也是同样的做法，但是需要搞清楚几个问题

#### 何时扩展窗口？

如果窗口中没有将所有的T中的字符包含，那么向右扩展窗口。

#### 何时更新长度？

扩展窗口之后，检查窗口的字符集合。如果当前窗口包含了所有的T中的字符，那么更新子串的起始位置id及长度。

#### 何时缩减窗口？

更新长度之后，窗口的左端右移，缩减窗口。于是有了下面的代码

```c++
// No. 76
string minWindow(string s, string t)
{
    int startId = 0;
    int rightId = 0;
    int minLen = s.length() + 1;
    int left = 0, right = 0;
    int  matchCount = 0;
    // record the only need characters
    unordered_map<char, int> window;
    unordered_map<char, int> need;

    for (auto ch : t)
        need[ch]++;
    
    while (right < s.length())
    {
        auto rch = s[right];
        right++;

        if (need.count(rch)) {
            // expand window
            window[rch]++;
            if (window[rch] == need[rch]) {
                matchCount++;
            }
        }       

        // shrink the window
        while (matchCount == need.size())
        {
            if (right - left < minLen)
            {
                startId = left;
                rightId = right;
                minLen = right - left;
            }

            auto lch = s[left];
            if (need.count(lch)) {
                window[lch]--;
                if (window[lch] < need[lch]) {
                    matchCount--;
                }
            }            
            left++;
        }
    }

    return rightId - startId > 0 ? s.substr(startId, rightId - startId) : "";
}
```

上面的写法中，需要注意几个问题

1. `window`仅仅记录了`[left, right)`的子串中的出现在need窗口中的字符的情况，并没有对窗口中所有的字符都做统计，其实也没有必要做，因为我们并不关心其他的字符；
2. `minLen`用来记录最短的子串长度，刚开始初始化为字符串的长度+1；
3. 我们用`matchCount`标记匹配的字符的个数，如果匹配了一个字符就加一，否则减一，匹配的标准是该字符的出现次数在window中**不少于**在need中出现的次数。注意，这个**参数在扩展窗口时增加，在缩减窗口时减少**，增加或减少之后即刻与need比较判断。

### 424. 替换后的最长重复字符
题目见[424. 替换后的最长重复字符 - 力扣（LeetCode）](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)，题目也比较简单，维护滑动窗口，每次在扩展窗口时候检查当前的窗口中的最多的字符的个数与替换的数量k的和是否大于等于窗口的长度？

1. 如果是，则更新窗口的长度，窗口继续扩展；
2. 如果否，则缩减窗口的长度；

循环往复，直到窗口的右边界超过了字符串的长度。代码如下

```c++
class Solution {
public:
    int characterReplacement(string s, int k) {
        int maxLen = 0;
        int left = 0, right = 0;
        unordered_map<char, int> window;
        while (right < s.length()) {
            auto rch = s[right];
            window[rch]++;
            right++;

            // find max count char
            int maxCount = -1;
            for (auto iter = window.begin(); iter != window.end(); iter++) {
                maxCount = max(maxCount, iter->second);
            }

            if (maxCount + k >= right - left) {
                maxLen = max(right - left, maxLen);
            } else {
                auto lch = s[left];
                window[lch]--;
                left++;
            }
        }

        return maxLen;
    }
};
```



### 1004. 最大连续1的个数 III
题目见[1004. 最大连续1的个数 III - 力扣（LeetCode）](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)，与上一题目类似，直接看代码

```c++
class Solution {
public:
    int longestOnes(vector<int>& A, int K) {
        int maxLen = 0;
        int left = 0, right = 0;
        unordered_map<int, int> window;
        while (right < A.size()) {
            auto rnum = A[right];
            window[rnum]++;
            right++;
                        
            if (window[1] + K >= right - left) {
                maxLen = max(right - left, maxLen);
            } else {
                auto lnum = A[left];
                window[lnum]--;
                left++;
            }
        }

        return maxLen;
    }
};
```

可以看出这个代码与上一题目非常相似，通用框架都是默认扩展窗口，如果不满足某些条件再缩减窗口。

### 992. K个不同整数的数组

题目见[992. K 个不同整数的子数组 - 力扣（LeetCode）](https://leetcode-cn.com/problems/subarrays-with-k-different-integers/)，这个比上面的两道题要复杂点，两次使用双指针解决，基本思路是：

1. 维护像之前一样的滑动窗口，如果当前的窗口不满足条件，那么向右扩张；
2. 如果窗口满足条件了，停止向右扩张，右边界不变，左边界向右移动（收缩窗口），开始计算满足条件的窗口数目。注意，这里的动作，在满足条件的窗口上再开一个滑动窗口，但是该窗口的右边界不变，不停右移左边界，遍历满足条件的总数；
3. 如此这般往复循环，直到滑动窗口的右边缘到达字符串的右边界为止。

我第一次提交的代码如下，

```c++
int subarraysWithKDistinct(vector<int>& A, int K) {
        int left = 0, right = 0;
        unordered_map<int, int> window;

        int count = 0;
        while (right < A.size()) {
            auto rnum = A[right];
            window[rnum]++;
            right++;
            
			// 如果当前的窗口中超过了K个不同的整数，那么需要缩小左边缘（即窗口左移）
            while (window.size() > K) {
                auto lnum = A[left];
                window[lnum]--;
                if (window[lnum] == 0) {
                    window.erase(lnum);
                }
                left++;
            } 
            
            // 当前的窗口有K个不同的整数，移动左边缘，遍历所有满足条件的窗口
            if (window.size() == K) {                
                unordered_map<int, int> subWindow(window);
                int tmpLeft = left;

                while (subWindow.size() == K) {
                    count++;
                    subWindow[A[tmpLeft]]--;
                    if (subWindow[A[tmpLeft]] == 0) {
                        subWindow.erase(A[tmpLeft]);
                    }
                    tmpLeft++;
                }
            }
        }

        return count;
    }
```

计算结果正确，但是超时了，仔细想想，其实不需要在创建一个subWindow窗口，可以复用原来的窗口，但是遍历完毕需要记得恢复，适当修改代码如下，

```c++
int subarraysWithKDistinct(vector<int>& A, int K) {
        int left = 0, right = 0;
        unordered_map<int, int> window;
        int count = 0;
        while (right < A.size()) {
            auto rnum = A[right];
            window[rnum]++;
            right++;

            while (window.size() > K) {
                auto lnum = A[left];
                window[lnum]--;
                if (window[lnum] == 0) {
                    window.erase(lnum);
                }
                left++;
            } 
            
             
            int tmpLeft = left;
            while (window.size() == K) {
                count++;
                window[A[tmpLeft]]--;
                if (window[A[tmpLeft]] == 0) {
                    window.erase(A[tmpLeft]);
                }
                tmpLeft++;
            }
            // recover the window
            while (tmpLeft > left) {
                window[A[tmpLeft-1]]++;
                tmpLeft--;
            }
        }
        return count;
    }
```

## 数据结构

### 哈希表

上面的示例都用到了哈希表，又称为散列表，具体的定义可以参考[哈希表](https://www.wikiwand.com/zh/%E5%93%88%E5%B8%8C%E8%A1%A8)。与一般的顺序访问的数组等数据结构不同，哈希表将查询的数据映射到表中的位置来记录，加快了查询的速度（类似于数组的下表和数组的值的映射关系）。一般而言，哈希表的查询，插入和删除的性能是$O(1)$。

### C++ STL 中的哈希表

C++在stl中使用`unordered_map`的数据结构保存哈希表，基本的用法如下所示

```c++
// C++ program to demonstrate functionality of unordered_map 
#include <iostream> 
#include <unordered_map>  // 
using namespace std; 

int main() 
{ 
	// Declaring umap to be of <string, int> type 
	// key will be of string type and mapped value will 
	// be of double type 
	unordered_map<string, int> umap; 

	// inserting values by using [] operator 
	umap["GeeksforGeeks"] = 10; 
	umap["Practice"] = 20; 
	umap["Contribute"] = 30; 

	// Traversing an unordered map 
	for (auto x : umap) 
	cout << x.first << " " << x.second << endl; 

} 
```

创建一个哈希表格，在哈希表中添加“键-值”对。

- `unordered_map`和`unordered_set`有什么区别？

  `unordered_set`中只保存了键，主要用于查看某元素是否在集合中，不能保存每个键出现的次数。

- `unordered_map`和`map`有什么区别？

  1. `map`中的键值是按序保存的，但是`unorederd_map`的键值是无序保存的；

  2. 二者底层实现的数据结构不同，`map`使用的是[红黑树](https://zh.wikipedia.org/zh-my/%E7%BA%A2%E9%BB%91%E6%A0%91)，操作性能分别是$O(logn)$和$O(1)$

常见的操作如下面的代码所示

```c++
// C++ program to demonstrate functionality of unordered_map 
#include <iostream> 
#include <unordered_map> 
using namespace std; 

int main()
{
    // Declaring umap to be of <string, double> type 
    // key will be of string type and mapped value will 
    // be of double type 
    unordered_map<string, double> umap;

    // 新增键值对
    umap["PI"] = 3.14;
    umap["root2"] = 1.414;
    umap["root3"] = 1.732;
    umap["log10"] = 2.302;
    umap["loge"] = 1.0;

    // 插入键值对，可以使用c++的make_pair函数
    umap.insert(make_pair("e", 2.718));

    string key = "PI";

    // 查询方法一
    if (umap.find(key) == umap.end())
        cout << key << " not found\n\n";

    // If key found then iterator to that key is returned 
    else
        cout << "Found " << key << "\n\n";

    // 查询方法二 
    if (umap.count(key) == 0 )
        cout << key << " not found\n\n";

    // If key found then iterator to that key is returned 
    else
        cout << "Found " << key << "\n\n";

    // 遍历方法一
    unordered_map<string, double>::iterator itr;
    cout << "\nAll Elements : \n";
    for (itr = umap.begin(); itr != umap.end(); itr++)
    {
        // itr works as a pointer to pair<string, double> 
        // type itr->first stores the key part and 
        // itr->second stroes the value part 
        cout << itr->first << " " << itr->second << endl;
    }

    // 遍历方法二
    for (auto iter : umap) {
        cout << iter.first << " " << iter.second << endl;
    }

    return 0;
}
```

需要格外注意，除了上面的`find`方法之外，查询某个键是否在哈希表中，还可以通过如下代码查询某个键值`key`

```c++
if (umap[key] > 0) {
    cout << key << " is found\n";
} else {
    cout << "Not found\n";
}
```

如果哈希表中没有这个键值，**那么它会自动添加进去，并赋给它初值**，所以在查询某个键是否存在时，不要用这样的方法。

## 参考资料

- [我写了首诗，把滑动窗口算法算法变成了默写题 - labuladong的算法小抄](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/hua-dong-chuang-kou-ji-qiao-jin-jie)，非常通俗易懂的算法说明，基本上按照这个顺序来刷题的
- [unordered_map in C++ STL - GeeksforGeeks](https://www.geeksforgeeks.org/unordered_map-in-cpp-stl/)，C++ stl中的unordered_map
- [Hashing | Set 1 (Introduction) - GeeksforGeeks](https://www.geeksforgeeks.org/hashing-set-1-introduction/)，哈希表的介绍
