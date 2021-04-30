# Adapter模式精解（C++版本）

本篇是设计模式第二篇，适配器模式，比较好理解。
<!--more-->

## 使用情景
> 适配器模式是一种**结构型**设计模式， 它能使接口**不兼容**的对象能够相互合作。

如果你有出国的经验，那么在出国前肯定会在淘宝买一个电源的转接插头带在身上，在国外旅行时为手机或者PC充电。为什么需要这个东西？参考这篇[国际旅行电源适配器指南](https://www.skyscanner.net/news/international-travel-plug-adapter-guide)，从文章中可以看出，每个不同的国家和地区的电源插座的形状和电压都不同，比如中国家用交流电是220V，而印度是230V，从电压的角度出发，你也需要一个东西将230V的电源转换成为稳定的220V，才能给电脑供电。

## 问题引入
那我们的问题自然就是如果我去印度旅行，如何使用工具将230V的电源转换成220V呢？
## 解决方案
解决方案也很简单，使用一个电源适配器即可，它负责将230V电源转换成为220V供我使用。
## UML表示及代码
参考《图解设计模式》的章节，我们有两种适配器模式。
### 基于继承的适配器模式

![类继承](https://pic.imgdb.cn/item/608ac9f7d1a9ae528ff52c22.png)

上面的图示中，`Banner`就是印度的230V电源，`PrintBanner`是电源适配器，`Print`表示我的电脑插头，`Main`函数是我自己。具体的C++代码如下所示
```cpp
#include <iostream>

class Banner {
public:
	Banner(std::string str = "") : str_(str) {}
	void showWithParen() const
	{
		std::cout << "("  << str_ << ")" <<  std::endl;
	}
	void showWithAster() const
	{
		std::cout << "*" << str_ << "*" << std::endl;
	}
private:
	std::string str_;
};

class Print {
public:
	virtual void printWeak() = 0;
	virtual void printStrong() = 0;
};

class PrintBanner : public Print, Banner {
public:
	PrintBanner(std::string str) : Banner(str) {}
	void printWeak() override {
		showWithParen();
	}
	void printStrong() override {
		showWithAster();
	}
};

using namespace std;

int main()
{
	auto printBanner = new PrintBanner("hello");
	printBanner->printWeak();
	printBanner->printStrong();

	delete printBanner;

	return 0;
}
```

### 基于委托的适配器模式
另外一种形式是基于委托的模式，这里的“委托”意思是我将本来需要我自己做的事情，交给别人来做，适配器`PrintBanner`将接口的功能**委托**给`Banner`去做。

![](https://pic.imgdb.cn/item/608ace89d1a9ae528f402db1.png)
具体的程序代码如下（与基于继承的代码仅仅在`PrintBanner`的类中的内容不同）
```cpp
class PrintBanner : public Print {
public:
	PrintBanner(std::string str = "") {
		banner_ = new Banner(str);
	}
	void printWeak() override {
		banner_->showWithParen();
	}
	void printStrong() override {
		banner_->showWithAster();
	}
	~PrintBanner() {
		if (banner_ != nullptr) {
			delete banner_;
		}
	}
private:
	Banner* banner_;
};
```

---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
