# Iterator模式精解（C++版本）


从本文章开始，参考结城浩的[《图解设计模式》](https://www.ituring.com.cn/book/1811)写23种设计模式的系列介绍文章，结合网络上其他的设计模式的内容，使用C++的语言编写这些设计模式，记录自己的设计模式的心得。

<!--more-->

## 设计模式分类

在开始设计模式的系列文章之前，先对设计模式的分类做一个说明，下面的说明摘选自[GoF 的 23 种设计模式的分类和功能](http://c.biancheng.net/view/1320.html)。

设计模式有两种分类方法，即根据模式的目的来分和根据模式的作用的范围来分。
### 根据目的来分
根据模式是用来完成什么工作来划分，这种方式可分为创建型模式、结构型模式和行为型模式 3 种。
- **创建型模式**：用于描述“怎样创建对象”，它的主要特点是“将对象的创建与使用分离”。GoF 中提供了单例、原型、工厂方法、抽象工厂、建造者等 5 种创建型模式。
- **结构型模式**：用于描述如何将类或对象按某种布局组成更大的结构，GoF 中提供了代理、适配器、桥接、装饰、外观、享元、组合等 7 种结构型模式。
- **行为型模式**：用于描述类或对象之间怎样相互协作共同完成单个对象都无法单独完成的任务，以及怎样分配职责。GoF 中提供了模板方法、策略、命令、职责链、状态、观察者、中介者、迭代器、访问者、备忘录、解释器等 11 种行为型模式。
###gg 根据作用范围来分
根据模式是主要用于类上还是主要用于对象上来分，这种方式可分为类模式和对象模式两种。
- 类模式：用于处理类与子类之间的关系，这些关系通过继承来建立，是静态的，在编译时刻便确定下来了。GoF中的工厂方法、（类）适配器、模板方法、解释器属于该模式。
- 对象模式：用于处理对象之间的关系，这些关系可以通过组合或聚合来实现，在运行时刻是可以变化的，更具动态性。GoF 中除了以上 4 种，其他的都是对象模式。

一般都选用第一种设计模式的分类方法，总结如下

![](https://pic.imgdb.cn/item/608bf193d1a9ae528f3a4b89.png)

## 使用情景
> 迭代器模式是一种行为设计模式， 提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象内部表示。

C++中比较典型类似与std的vector的前向迭代器`begin()`和`end()`，以及后向迭代器`rbegin()`和`rend()`。

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

## C++代码
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
    virtual int Count() const = 0;
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
这个模式有实际上由两个角色，**集合以及集合的迭代器**，这两个事物由分为抽象和具体两种。我们所举的例子中
- `BookShelf`就是具体的集合，它实现自抽象的集合接口`Aggregate`；
- `BookShelfIterator`是具体的迭代器，它实现自抽象的迭代器接口`Iterator`。
迭代器接口中定义了迭代器的遍历的所有方法，`HasNext`以及`Next`。

这里需要注意的是，在C++的版本里面集合的接口中不仅定义了创建迭代器的方法`CreateIterator`，而且定义了集合的个数`Count`和集合获取当前元素的方法`getBookAt`，这是因为迭代器中的方法实现依赖这些接口。

## 扩展修改

大家可以思考一下为什么我们需要这个模式呢？设计模式的初衷是实现代码的**复用和可扩展**，这个模式体现在哪里呢？

### 遍历方法和集合本身解耦

迭代器模式的重要作用是**将集合的遍历和实现分离开来**，换句话说，无论实现如何变化，我依然可以使用原来的方法进行遍历，也就是说**遍历这个动作本身**不会因为书架的实现发生变化而变化，所以下面的代码是不变的
```cpp
while (iter->HasNext() ==  true) {
		cout << iter->Next().getName() << endl;
	}
```
我们依然仅仅依靠`HasNext`以及`Next`接口就可以完成对于书架本身的遍历。

###  遍历需求易于扩展

不妨设想一下，假如我们需要在原来的书架上新增一种遍历方式，从后向前实现后向遍历，这个代码该怎么修改呢？应该完成如下的工作：

1. 在原来的书架类中新增后向遍历的iter指针；
2. 书架类中包含创建后向迭代器的方法；
3. 后向迭代器继承自迭代器类别，实现后向迭代器

然后就可以继续使用原来的遍历方式遍历集合了，具体的代码如下
```cpp

enum State {
    RIGHT = 0,
    WRONG,
    RESERVED = 22
};
// concrete book
class Book {
public:
    Book(std::string name = "") : name_(name) {};
    std::string getName() const { return name_; };
    ~Book() = default;
private:
    std::string name_;
};


class Iterator {
public:
    virtual Book Next() = 0;
    virtual bool HasNext() const = 0;
    virtual ~Iterator() = default;
};


class Aggregate {
public:
    virtual Iterator* CreateForwardIterator() = 0;
    virtual Iterator* CreateBackwardIterator() = 0;
    virtual State getBookAt(const int index, Book& book) = 0;
    virtual int Count() const = 0;
    virtual ~Aggregate() = default;
};

class BookShelfBackwardIterator : public Iterator {
public:
    BookShelfBackwardIterator(Aggregate* aggregate) : aggregate_(aggregate) {
        loc_ = aggregate_->Count() - 1;
    };
    ~BookShelfBackwardIterator() {
        if (aggregate_ != nullptr) {
            aggregate_ = nullptr;
        }
        loc_ = -1;
    }
    Book Next() {
        Book book;
        aggregate_->getBookAt(loc_, book);
        loc_--;
        return book;
    }

    bool HasNext() const {
        return loc_ >= 0;
    }
private:
    int loc_;
    Aggregate* aggregate_;
};

class BookShelfForwardIterator : public Iterator {
public:
    BookShelfForwardIterator(Aggregate* aggregate) : aggregate_(aggregate), loc_(0) {};
    ~BookShelfForwardIterator() {
        if (aggregate_ != nullptr) {
            aggregate_ = nullptr;
        }
        loc_ = -1;
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

class BookShelf : public Aggregate {
public:
    BookShelf(const int maxSize) :maxSize_(maxSize), count_(0), iterator_(nullptr), backiter_(nullptr) {
        books_.clear();
    }

    Iterator* CreateBackwardIterator() {
        if (backiter_ == nullptr) {
            backiter_ = new BookShelfBackwardIterator(this);
        }
        return backiter_;
    }

    Iterator* CreateForwardIterator() {
        if (iterator_ == nullptr) {
            iterator_ = new BookShelfForwardIterator(this);
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

        if (backiter_ != nullptr) {
            delete backiter_;
            backiter_ = nullptr;
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
    Iterator* backiter_;
};

using namespace std;
// book class

int main()
{
	BookShelf* myShelf = new BookShelf(5);
	myShelf->Push(Book("《重构》"));
	myShelf->Push(Book("《图解设计模式》"));
	myShelf->Push(Book("《黎曼猜想》"));

	Iterator* iter = myShelf->CreateForwardIterator();
	cout << "前向遍历" << myShelf->Count() << "本书：\n";
	while (iter->HasNext()) {
		cout << iter->Next().getName() << endl;
	}
	iter = nullptr;

	iter = myShelf->CreateBackwardIterator();
	cout << "后向遍历" << myShelf->Count() << "本书：\n";
	while (iter->HasNext()) {
		cout << iter->Next().getName() << endl;
	}

	return 0;
}
```
运行效果如下
```shell
前向遍历3本书：
《重构》
《图解设计模式》
《黎曼猜想》
后向遍历3本书：
《黎曼猜想》
《图解设计模式》
《重构》
```

## 参考资料
1. [图说设计模式 — Graphic Design Patterns](https://design-patterns.readthedocs.io/zh_CN/latest/)
2. [图灵程序设计丛书 图解设计模式](https://edu.heibai.org/%E5%9B%BE%E8%A7%A3%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.pdf)，图解设计模式的pdf下载地址
3. [设计模式目录：22种设计模式](https://refactoringguru.cn/design-patterns/catalog)，乌克兰的某个程序员写的设计模式的网页，非常不错
---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
