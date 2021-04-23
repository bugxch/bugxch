# Iterator（迭代器）模式

从本文章开始，参考结城浩的[《图解设计模式》](https://www.ituring.com.cn/book/1811)写23种设计模式的系列介绍文章，结合网络上其他的设计模式的内容，使用C++的语言编写这些设计模式，记录自己的设计模式的心得。

<!--more-->

## 使用情景
迭代器的作用是遍历某一类相同元素的集合，所以迭代器模式是一种**行为**设计模式，该模式的作用是在不改变底层元素的前提下，按照某种方式遍历集合中元素的值。其中，在C++中比较典型类似与std的vector的前向迭代器`begin()`和`end()`，以及后向迭代器`rbegin()`和`rend()`。

## 问题引入
可以设想元素聚合成集合的方式，最简单的方式是顺序存储的数组或者列表，但是也有例如树、图和其他复杂的数据结构。所以，如果要遍历集合中的元素，有下面的两个问题需要解决：

1. 如何在不改变元素的前提下，遍历各种不同的集合？
2. 同一个集合如果有不同的遍历方式（比如树有前中后序3种不同的遍历顺序），如果在不改变集合的前提下，比较容易扩展这些功能呢？

![树的深度和广度遍历](https://refactoringguru.cn/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c8)

## 解决方案

为了满足开闭原则，我们将定义一个迭代器的类，将迭代从集合种抽象出来，作为一个行为的迭代器处理。如下图所示，对于一个树结构，我们定义了两个迭代器的类，DFS和BFS两个迭代器，这样可以将元素遍历和树本身进行解耦。

![](https://refactoringguru.cn/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2)

## UML表示
我们以《图解设计模式》中遍历书架中的书本为例，有如下的UML图

![UML图示](https://pic.imgdb.cn/item/6079a1d38322e6675cca8bee.png)

对上面的这幅图做进一步的说明，
- Aggregate表示集合的接口，书架实现该接口，所以书架必须要有迭代器的方法；
- Iterator表示集合的迭代器的接口，书架的迭代器实现它；
需要说明的是，此处的迭代器只有一种前向的迭代器，也可以定义后向遍历的迭代器。

## 示例代码
我们使用C++按照上面的UML图片实现这个设计模式。
```cpp
#include <iostream>
using namespace std;

enum State {
    RIGHT = 0,
    WRONG,
    RESERVED = 22
};
// 书本的类
class Book {
public:
    Book(std::string name = "") : name_(name) {};
    std::string getName() const { return name_; };
    ~Book() = default;
private:
    std::string name_;
};

// 抽象的迭代器类，包括获得下一本书，以及是否有下一本书的判断
class Iterator {
public:
    virtual Book Next() = 0;
    virtual bool HasNext() const = 0;
    virtual ~Iterator() = default;
};

// 抽象的聚合类，该类有创建迭代器、取得某个位置的书本，弹出书本，计数，加入书本等功能
class Aggregate {
public:
    virtual Iterator* CreateIterator() = 0;
    virtual State getBookAt(const int index, Book& book) = 0;
    virtual void Pop() = 0;
    virtual int Count() const = 0;
    virtual void Push(const Book& book) = 0;
    virtual ~Aggregate() = default;
};

// 书架的具体迭代器，实现上面的抽象类的虚函数
class BookShelfIterator : public Iterator {
public:
    BookShelfIterator(Aggregate* aggregate) : aggregate_(aggregate), loc_(0) {};
    ~BookShelfIterator() {
        if (aggregate_ != nullptr) {
            delete aggregate_;
            aggregate_ = nullptr;
        }
        loc_ = 0;
    }
    Book Next() {
        Book book;
        aggregate_->getBookAt(loc_, book);
        loc_++;
        return book;
    }

    bool HasNext() const {
        return loc_ < aggregate_->Count();
    }
private:
    int loc_;
    Aggregate* aggregate_;
};

// 具体的聚合类——书架，实现上面的功能
class BookShelf : public Aggregate {
public:
    BookShelf(const int maxSize) :maxSize_(maxSize), count_(0), iterator_(nullptr) {
        books_.clear();
    }

    Iterator* CreateIterator() {
        if (iterator_ == nullptr) {
            iterator_ = new BookShelfIterator(this);
        }
        return iterator_;
    }

    State getBookAt(const int index, Book& book) {
        if (index >= count_) {
            std::cout << "Wrong index\n";
            return WRONG;
        }
        book = books_[index];
        return RIGHT;
    }

    void Pop() {
        books_.pop_back();
        count_--;
    }

    int Count() const {
        return count_;
    }

    void Push(const Book& book) {
        if (count_ == maxSize_) {
            std::cout << "bookshelf is full\n";
            return;
        }
        books_.push_back(book);
        count_++;
    }
    ~BookShelf() {
        if (iterator_ != nullptr) {
            delete iterator_;
            iterator_ = nullptr;
        }
        maxSize_ = 0;
        count_ = 0;
        books_.clear();
    }
private:
    int maxSize_;
    int count_;
    std::vector<Book> books_;
    Iterator* iterator_;
};

// client
int main()
{
	BookShelf* myShelf = new BookShelf(5);
	myShelf->Push(Book("《重构》"));
	myShelf->Push(Book("《图解设计模式》"));
	myShelf->Push(Book("《黎曼猜想》"));

	Iterator* iter = myShelf->CreateIterator();
	cout << "书架上有" << myShelf->Count() << "本书：\n";
	while (iter->HasNext() ==  true) {
		cout << iter->Next().getName() << endl;
	}
	return 0;
}
```
具体的运行结果如下
```shell
书架上有3本书：
《重构》
《图解设计模式》
《黎曼猜想》
```
## 分析角色


## 参考资料
1. [图说设计模式 — Graphic Design Patterns](https://design-patterns.readthedocs.io/zh_CN/latest/)
2. [图灵程序设计丛书 图解设计模式](https://edu.heibai.org/%E5%9B%BE%E8%A7%A3%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.pdf)，图解设计模式的pdf下载地址
3. [设计模式目录：22种设计模式](https://refactoringguru.cn/design-patterns/catalog)，乌克兰的某个程序员写的设计模式的网页，非常不错
---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
