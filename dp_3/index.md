# Template Method模式精解（C++版本）

设计模式第三弹，设计模式**行为型模式**中的模板方法，也比较简单。

<!--more-->
## 使用情景
> 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Template Method是行为型模式，使得子类可以不改变算法的结构（步骤）即可重定义该算法的某些特定步骤。

模板方法将整个算法转换为一系列独立的步骤， 以便子类能对其进行扩展， 同时还可让超类中所定义的结构保持完整。可以设想我们上学时候临摹毛笔字，你可以使用墨汁沿着田字格中的汉字临摹，也可以使用红墨水临摹，无论用哪种颜色的墨水，最后完成的字的形状是一样的。

![图片来自网络](https://pic.imgdb.cn/item/608bf40bd1a9ae528f53b668.png)

汉字的字形就是模板，每个学生使用不同的工具或者墨水按照模板习字，就是模板方法。

## 问题引入

Template Method就是带有模板功能的模式，它有下面的特点：
1. 组成模板的方法被定义在父类中，但是这些方法是抽象方法，具体的方法实现由各个子类实现；
2. 父类中定义了**处理流程的框架**，这个流程由上面定义的这些方法**按照特定的步骤完成**。

打一个比方，如果我们村里的每个人盖一座房子，无论是谁都需要完成如下的步骤，准备材料，设计图纸，雇佣施工队，开工建设，完成这些步骤之后才能盖起一座完整的房子。但是不同的人使用的材料不同，设计的图纸不同，施工队的质量也不一样，依照主人的品味和资金实力每一个步骤不同的人做就有不同的效果。这里的所有步骤就是模板方法，不同的人就是子类。

## UML表示及代码

参考《图解设计模式》中第三章的例子，UML图及代码如下所示

![](https://pic.imgdb.cn/item/608bf6c0d1a9ae528f6ea0ee.png)
每个类的作用如下
![](https://pic.imgdb.cn/item/608bf701d1a9ae528f712d27.png)

- `AbstractDisplay`是抽象类，定义了整个的**流程框架**，即方法`display()`，该方法又由3个抽象方法实现`open(), print(), close()`；
- `CharDisplay`和`StringDisplay`是具体的继承类，它们实现了抽象类中的抽象方法。
仅仅从抽象类看不出来每个抽象方法的具体实现，这些方法由每个类具体负责，上面的所有的类的具体代码如下。

```cpp
#include <string>
#include <iostream>
using namespace std;
class AbstractDisplay {
public:
	virtual void open() = 0;
	virtual void print() = 0;
	virtual void close() = 0;
	virtual void display() final
	{
		open();
		for (int i = 0; i < 5; i++) {
			print();
		}
		close();
	}
};

class CharDisplay: public AbstractDisplay {
public:
	CharDisplay(char ch = 'h') : ch_(ch) {};
	void open() override
	{
		cout << "<<";
	}
	void close() override
	{
		cout << ">>\n";
	}
	void print() override
	{
		cout << ch_;
	}
private:
	char ch_;
};

class StringDisplay : public AbstractDisplay {
public:
	StringDisplay(string str = " ", int width = 10) :width_(width), str_(str) {};
	void open() override
	{
		printLine();
	}
	void print() override
	{
		cout << "|" << str_ << "|\n";
	}
	void close() override
	{
		printLine();
	}
private:
	string str_;
	int width_;
	void printLine() const {
		cout << "+";
		for (int i = 0; i < width_; i++) {
			cout << "-";
		}
		cout << "+\n";
	}
};

int main()
{
	AbstractDisplay* display = new CharDisplay('H');
	display->display();
	delete display;
	display = new StringDisplay("Hello World!");
	display->display();
	delete display;
	display = new StringDisplay("Hello haha!");
	display->display();
	delete display;

	return 0;
}
```
运行结果如下，
```shell
<<HHHHH>>
+----------+
|Hello World!|
|Hello World!|
|Hello World!|
|Hello World!|
|Hello World!|
+----------+
+----------+
|Hello haha!|
|Hello haha!|
|Hello haha!|
|Hello haha!|
|Hello haha!|
+----------+
```

---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
