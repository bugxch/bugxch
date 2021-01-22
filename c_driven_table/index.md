# [转载]表驱动法代码实践

这是一篇转载，点击查看[原文链接](https://www.cnblogs.com/clover-toeic/p/3730362.html)。
<!--more-->
> *数据压倒一切。如果选择了正确的数据结构并把一切组织的井井有条，正确的算法就不言自明。编程的核心是数据结构，而不是算法. -- Rob Pike*

#   说明
本文基于这样的认识：数据是易变的，逻辑是稳定的。
本文例举的编程实现多为代码片段，但不影响描述的完整性。
本文例举的编程虽然基于C语言，但其编程思想也适用于其他语言。
此外，本文不涉及语言相关的运行效率讨论。

# 概念提出
所谓表驱动法(Table-Driven Approach)简而言之就是用查表的方法获取数据。此处的“表”通常为数组，但可视为数据库的一种体现。

根据字典中的部首检字表查找读音未知的汉字就是典型的表驱动法，即以每个字的字形为依据，计算出一个索引值，并映射到对应的页数。相比一页一页地顺序翻字典查字，部首检字法效率极高。

具体到编程方面，在数据不多时可用逻辑判断语句(if…else或switch…case)来获取值；但随着数据的增多，逻辑语句会越来越长，此时表驱动法的优势就开始显现。

例如，用36进制(A表示10，B表示11，…)表示更大的数字，逻辑判断语句如下：
```cpp
if(ucNum < 10)
{
    ucNumChar = ConvertToChar(ucNum);
}
else if(ucNum == 10)
{
    ucNumChar = 'A';
}
else if(ucNum == 11)
{
    ucNumChar = 'B';
}
else if(ucNum == 12)
{
    ucNumChar = 'C';
}
//... ...
else if(ucNum == 35)
{
    ucNumChar = 'Z';
}
```
当然也可以用switch…case结构，但实现都很冗长。而用表驱动法(将numChar存入数组)则非常直观和简洁。如：
```cpp
CHAR aNumChars[] = {'0', '1', '2', /*3~9*/'A', 'B', 'C', /*D~Y*/'Z'};
CHAR ucNumChar = aNumChars[ucNum % sizeof(aNumChars)];
```
像这样直接将变量当作下数组下标来读取数值的方法就是直接查表法。注意，如果熟悉字符串操作，则上述写法可以更简洁：
```cpp
CHAR ucNumChar = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"[ucNum];
```
使用表驱动法时需要关注两个问题：一是如何查表，从表中读取正确的数据；二是表里存放什么，如数值或函数指针。前者参见1.1节“查表方式”内容，后者参见1.2节“实战示例”内容。

## 查表方式
常用的查表方式有直接查找、索引查找和分段查找等。
### 直接查找
即直接通过数组下标获取到数据。如果熟悉哈希表的话，可以很容易看出这种查表方式就是哈希表的直接访问法。

如获取星期名称，逻辑判断语句如下：
```cpp
if(0 == ucDay)
{
    pszDayName = "Sunday";
}
else if(1 == ucDay)
{
    pszDayName = "Monday";
}
//... ...
else if(6 == ucDay)
{
    pszDayName = "Saturday";
}
```
而实现同样的功能，可将这些数据存储到一个表里：
```cpp
CHAR *paNumChars[] = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday",  "Saturday"};
CHAR *pszDayName = paNumChars[ucDay];
```
类似哈希表特性，表驱动法适用于无需有序遍历数据，且数据量大小可提前预测的情况。

对于过于复杂和庞大的判断，可将数据存为文件，需要时加载文件初始化数组，从而在不修改程序的情况下调整里面的数值。

有时，访问之前需要先进行一次键值转换。如表驱动法表示端口忙闲时，需将槽位端口号映射为全局编号。所生成的端口数目大小的数组，其下标对应全局端口编号，元素值表示相应端口的忙闲状态。
### 索引查找

有时通过一次键值转换，依然无法把数据(如英文单词等)转为键值。此时可将转换的对应关系写到一个索引表里，即索引访问。

如现有100件商品，4位编号，范围从0000到9999。此时只需要申请一个长度为100的数组，且对应2位键值。但将4位的编号转换为2位的键值，可能过于复杂或没有规律，最合适的方法是建立一个保存该转换关系的索引表。采用索引访问既节省内存，又方便维护。比如索引A表示通过名称访问，索引B表示通过编号访问。
### 分段查找
通过确定数据所处的范围确定分类(下标)。有的数据可分成若干区间，即具有阶梯性，如分数等级。此时可将每个区间的上限(或下限)存到一个表中，将对应的值存到另一表中，通过第一个表确定所处的区段，再由区段下标在第二个表里读取相应数值。注意要留意端点，可用二分法查找，另外可考虑通过索引方法来代替。

如根据分数查绩效等级：
```cpp
#define MAX_GRADE_LEVEL   (INT8U)5
DOUBLE aRangeLimit[MAX_GRADE_LEVEL] = {50.0, 60.0, 70.0, 80.0, 100.0};
CHAR *paGrades[MAX_GRADE_LEVEL] = {"Fail", "Pass", "Credit", "Distinction", "High Distinction"};

static CHAR* EvaluateGrade(DOUBLE dScore)
{
    INT8U ucLevel = 0;
    for(; ucLevel < MAX_GRADE_LEVEL; ucLevel++)
    {
        if(dScore < aRangeLimit[ucLevel])
            return paGrades[ucLevel];
    }
    return paGrades[0];
}
```
上述两张表(数组)也可合并为一张表(结构体数组)，如下所示：
```cpp
typedef struct{
    DOUBLE  aRangeLimit;
    CHAR    *pszGrade;
}T_GRADE_MAP;

T_GRADE_MAP gGradeMap[MAX_GRADE_LEVEL] = {
    {50.0,              "Fail"},
    {60.0,              "Pass"},
    {70.0,              "Credit"},
    {80.0,              "Distinction"},
    {100.0,             "High Distinction"}
};

static CHAR* EvaluateGrade(DOUBLE dScore)
{
    INT8U ucLevel = 0;
    for(; ucLevel < MAX_GRADE_LEVEL; ucLevel++)
    {
        if(dScore < gGradeMap[ucLevel].aRangeLimit)
            return gGradeMap[ucLevel].pszGrade;
    }
    return gGradeMap[0].pszGrade;
}
```
该表结构已具备的数据库的雏形，并可扩展支持更为复杂的数据。其查表方式通常为索引查找，偶尔也为分段查找；当索引具有规律性(如连续整数)时，退化为直接查找。

使用分段查找法时应注意边界，将每一分段范围的上界值都考虑在内。找出所有不在最高一级范围内的值，然后把剩下的值全部归入最高一级中。有时需要人为地为最高一级范围添加一个上界。

同时应小心不要错误地用“<”来代替“<=”。要保证循环在找出属于最高一级范围内的值后恰当地结束，同时也要保证恰当处理范围边界。

## 实战示例
本节多数示例取自实际项目。表形式为一维数组、二维数组和结构体数组；表内容有数据、字符串和函数指针。基于表驱动的思想，表形式和表内容可衍生出丰富的组合。
### 字符统计
问题：统计用户输入的一串数字中每个数字出现的次数。普通解法主体代码如下：
```cpp
INT32U aDigitCharNum[10] = {0}; /* 输入字符串中各数字字符出现的次数 */
INT32U dwStrLen = strlen(szDigits);

INT32U dwStrIdx = 0;
for(; dwStrIdx < dwStrLen; dwStrIdx++)
{
    switch(szDigits[dwStrIdx])
    {
        case '1':
            aDigitCharNum[0]++;
            break;
        case '2':
            aDigitCharNum[1]++;
            break;
        //... ...
        case '9':
            aDigitCharNum[8]++;
            break;
    }
}
```
这种解法的缺点显而易见，既不美观也不灵活。其问题关键在于未将数字字符与数组aDigitCharNum下标直接关联起来。

以下示出更简洁的实现方式：
```cpp
for(; dwStrIdx < dwStrLen; dwStrIdx++)
{
    aDigitCharNum[szDigits[dwStrIdx] - '0']++;
}
```
上述实现考虑到0也为数字字符。该解法也可扩展至统计所有ASCII可见字符。

【待续】



---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
