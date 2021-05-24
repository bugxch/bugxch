# C++中的sort函数详解


公司认证的leetcode题目中经常会用到sort函数，不是很熟悉，今天系统学习总结下。

<!--more-->

## 总述

下面是C++的stl中的排序的所有函数，这个系列的博客会逐一介绍，这次的博客先关注最常用的`sort`函数。

| 函数名                                                     | 用法                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| sort (first, last)                                         | 对容器或普通数组中 [first, last) 范围内的元素进行排序，默认进行升序排序。 |
| stable_sort (first, last)                                  | 和 sort() 函数功能相似，不同之处在于，对于 [first, last) 范围内值相同的元素，该函数不会改变它们的相对位置。 |
| partial_sort (first, middle, last)                         | 从 [first,last) 范围内，筛选出 muddle-first 个最小的元素并排序存放在 [first，middle) 区间中。 |
| partial_sort_copy (first, last, result_first, result_last) | 从 [first, last) 范围内筛选出 result_last-result_first 个元素排序并存储到 [result_first, result_last) 指定的范围中。 |
| is_sorted (first, last)                                    | 检测 [first, last) 范围内是否已经排好序，默认检测是否按升序排序。 |
| is_sorted_until (first, last)                              | 和 is_sorted() 函数功能类似，唯一的区别在于，如果 [first, last) 范围的元素没有排好序，则该函数会返回一个指向首个不遵循排序规则的元素的迭代器。 |
| void nth_element (first, nth, last)                        | 找到 [first, last) 范围内按照排序规则（默认按照升序排序）应该位于第 nth 个位置处的元素，并将其放置到此位置。同时使该位置左侧的所有元素都比其存放的元素小，该位置右侧的所有元素都比其存放的元素大。 |

## sort函数

### 使用范围

C++ STL 标准库中的 sort() 函数，本质就是一个模板函数。正如表 1 中描述的，该函数专门用来对容器或普通数组中指定范围内的元素进行排序，排序规则默认以元素值的大小做升序排序，除此之外我们也可以选择标准库提供的其它排序规则（比如`std::greater<T>`降序排序规则），甚至还可以自定义排序规则。

需要注意的是，sort() 函数受到底层实现方式的限制，它仅适用于普通数组和部分类型的容器。换句话说，只有普通数组和具备以下条件的容器，才能使用 sort() 函数：

1. 容器支持的迭代器类型必须为随机访问迭代器。这意味着，sort() 只对 array、vector、deque 这 3 个容器提供支持；
2. 如果对容器中指定区域的元素做默认升序排序，则元素类型必须支持`<`小于运算符；同样，如果选用标准库提供的其它排序规则，元素类型也必须支持该规则底层实现所用的比较运算符；
3. sort() 函数在实现排序时，需要交换容器中元素的存储位置。这种情况下，如果容器中存储的是自定义的类对象，则该类的内部必须提供移动构造函数和移动赋值运算符。

### 局限

`sort`函数不保证排序的[稳定性](https://baike.baidu.com/item/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95%E7%A8%B3%E5%AE%9A%E6%80%A7)，即如果被排序的序列中有多个相同值的元素，并不能保证排序之后他们的相对位置保持不变。

### 使用方法

值得一提的是，sort() 函数位于`<algorithm>`头文件中，因此在使用该函数前，程序中应包含如下语句：

```
#include <algorithm>
```

sort() 函数有 2 种用法，其语法格式分别为：

```cpp
//对 [first, last) 区域内的元素做默认的升序排序
void sort (RandomAccessIterator first, RandomAccessIterator last);
//按照指定的 comp 排序规则，对 [first, last) 区域内的元素进行排序
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

其中，first 和 last 都为随机访问迭代器，它们的组合 [first, last) 用来指定要排序的目标区域；另外在第 2 种格式中，comp 可以是 C++ STL 标准库提供的排序规则（比如 std::greater<T>），也可以是自定义的排序规则。比如，如果需要做**降序**排序，那么可以使用`std::less<T>`，也可以自己写一个降序的函数。具体的用法如下所示

```cpp
#include <iostream>     // std::cout
#include <algorithm>    // std::sort
#include <vector>       // std::vector
//以普通函数的方式实现自定义排序规则
bool mycomp(int i, int j) {
    return (i < j);
}
//以函数对象的方式实现自定义排序规则
class mycomp2 {
public:
    bool operator() (int i, int j) {
        return (i < j);
    }
};

int main() {
    std::vector<int> myvector{ 32, 71, 12, 45, 26, 80, 53, 33 };
    //调用第一种语法格式，对 32、71、12、45 进行排序
    std::sort(myvector.begin(), myvector.begin() + 4); //(12 32 45 71) 26 80 53 33
    //调用第二种语法格式，利用STL标准库提供的其它比较规则（比如 greater<T>）进行排序
    std::sort(myvector.begin(), myvector.begin() + 4, std::greater<int>()); //(71 45 32 12) 26 80 53 33

    //调用第二种语法格式，通过自定义比较规则进行排序
    std::sort(myvector.begin(), myvector.end(), mycomp2()); // 80 71 53 45 33 32 26 12
    sort(myvector.begin() + 4, myvector.end(), mycomp); // 80 71 53 45 12 26 32 33
    //输出 myvector 容器中的元素
    for (std::vector<int>::iterator it = myvector.begin(); it != myvector.end(); ++it) {
        std::cout << *it << ' ';
    }
    return 0;
}
```
## 再探自定义比较函数

### 一元谓词和二元谓词

sort的自定义比较函数在C++中成为**谓词**，在泛型编程中作为参数使用。按照接受参数的个数不同，谓词分为一元谓词和二元谓词两种。

- 一元谓词，比如`for_each`中使用，因为该算法是顺序遍历容器中的每个元素，对每个元素进行操作，所以是一元谓词，如下面的代码片段

  ```cpp
  vector<string> str = { "i", "love", "leetcode", "i", "love", "coding" };
  void printEle(string str)
  {
      cout << str << " ";
  }
  for_each(str.begin(), str.end(), printEle) // printEle是一元谓词
  ```

- 二元谓词，sort算法是对容器的两个元素进行比较，所以接受两个参数，比如上面的`mycomp`函数。

### 可调用对象

谓词就是一个可调用对象(callable object)，在C++中可调用对象包括4种类型：函数、函数指针、重载函数调用符的类（可以像函数一样使用的类）以及**lambda表达式**。其实在上面的代码片段中，已经在sort算法中使用过函数以及重载函数调用符的类。此处重点介绍一下lambda表达式。lambda表达式的介绍很多，此处直接贴出来参考资料3中的总结表格

![](https://pic.imgdb.cn/item/60a9bec435c5199ba7e61a53.png)

从表格中可以看出捕获的类型，分为不捕获局部变量、按值捕获、按引用捕获，混合捕获这几种。参考[std::sort参考手册](https://zh.cppreference.com/w/cpp/algorithm/sort)中的代码，`sort`的用法如下所示

```cpp
#include <algorithm>
#include <functional>
#include <array>
#include <iostream>
 
int main()
{
    std::array<int, 10> s = {5, 7, 4, 2, 8, 6, 1, 9, 0, 3}; 
 
    // 用默认的 operator< 排序
    std::sort(s.begin(), s.end());
    for (auto a : s) {
        std::cout << a << " ";
    }   
    std::cout << '\n';
 
    // 用标准库比较函数对象排序
    std::sort(s.begin(), s.end(), std::greater<int>());
    for (auto a : s) {
        std::cout << a << " ";
    }   
    std::cout << '\n';
 
    // 用自定义函数对象排序
    struct {
        bool operator()(int a, int b) const
        {   
            return a < b;
        }   
    } customLess;
    std::sort(s.begin(), s.end(), customLess);
    for (auto a : s) {
        std::cout << a << " ";
    }   
    std::cout << '\n';
 
    // 用 lambda 表达式排序
    std::sort(s.begin(), s.end(), [](int a, int b) {
        return b < a;   
    });
    for (auto a : s) {
        std::cout << a << " ";
    } 
    std::cout << '\n';
}
```

输出

```
0 1 2 3 4 5 6 7 8 9 
9 8 7 6 5 4 3 2 1 0 
0 1 2 3 4 5 6 7 8 9 
9 8 7 6 5 4 3 2 1 0
```

表示了3种谓词，标准库、函数对象和lambda表达式。这里的二元谓词，告诉了`sort`，当比较**其中两个元素**的时候该如何处理两个元素的位置。

```cpp
struct {
        bool operator()(int a, int b) const
        {   
            return a < b;
        }   
    } customLess;
```

当上面的函数返回为`true`时候，那么将`a`排在`b`的前面，上面的代码种当`a < b`时结果为`true`，所以小的元素排在前面，下面通过做题来示例它的用法。

### 具体题目

参考[1366. 通过投票对团队排名 - 力扣（LeetCode）](https://leetcode-cn.com/problems/rank-teams-by-votes/)，具体的代码如下

```cpp
class Solution {
public:
	Solution() = default;
	string rankTeams(vector<string>& votes) 
	{
		unordered_map<char, vector<int>> ranks;

		for (auto& vote : votes[0]) {
			ranks[vote].resize(votes[0].size());
		}

		for (auto& vote : votes) {
			for (int i = 0; i < vote.size(); i++)
			{
				ranks[vote[i]][i]++;
			}
		}

		using PCV = pair<char, vector<int>>;
		vector<PCV> ranking;
		for (auto iter = ranks.begin(); iter != ranks.end(); iter++) {
			ranking.push_back({ iter->first, iter->second });
		}
		
        // lambda表达式
		sort(ranking.begin(), ranking.end(), [](PCV& a, PCV& b)
            {
                int i = 0;
                while (i < a.second.size()) {
                    if (a.second[i] != b.second[i]) {
                        return a.second[i] > b.second[i];
                    }
                    i++;
                }
                return a.first < b.first;
            });
		string res;
		for (int i = 0; i < ranking.size(); i++) {
			res += ranking[i].first;
		}
		return res;
	}
};
```

以题目中的示例1分析题意

|      | 第一名得票 | 第二名得票 | 第三名得票 |
| :--: | :--------: | :--------: | :--------: |
|  A   |     5      |     0      |     2      |
|  B   |     0      |     2      |     3      |
|  C   |     0      |     3      |     0      |

有5个人投票，如果给ABC的3人，从第一名到第三名依次唱票，

- 如果第一名决出胜者，那么该选手获得第一名，剩下的选手角逐第二名；
- 如果第二名决出胜者，那么该选手获得第一名，剩下的选手就是第三名。

如果参选人数超过3人，那么依此类推，直到所有名次所有人都占用为止。这里有一种特殊情况，如果有若干人获得相同的名次，那么以人名的字母排序。比如，如果A和B都得了第一名，那么排序A在前，B在后。注意上面的26行~36行的代码。它表示从第一名到最后一名排序，

- 如果两个选手的在第`i`个名次上票数相同，那么在第`i`个名次上不做任何操作（我们认为他们的名次是不分先后的），继续下一个名次`i++`的比较（第33行）；
- 如果两个选手在第`i`个名次上票数不同，那么以票数多者优先排序，退出循环，后面的名次不需要再比较了（第31行）；
- 如果在两个选手在所有的名次上票数均相同，那么最后按照人名排序（第35行）

这里的代码告诉了`sort`函数该如何对当前所有选手中的两个选手的名次进行排序，它会将其中的两两进行比较给出答案（如何两两比较，我们不用关心），**从微观层面告诉`sort`函数的两个元素的操作方法**，它就能将所有的选手按照这个方法排好序，这个就是lambda表达式的意义。

## 参考资料

1. [C++ sort()排序函数用法详解](http://c.biancheng.net/view/7457.html)，c语言中文网的介绍
2. [std::sort() in C++ STL - GeeksforGeeks](https://www.geeksforgeeks.org/sort-c-stl/)，国外的网站介绍
3. [C++ Primer 中文版](https://book.douban.com/subject/25708312/)中的10.3节
