# Factory Method模式精解（C++版本）

继续之前的设计模式第四弹，这次是大名鼎鼎的工厂方法模式。
<!--more-->
## 使用情景
> 定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method是创建型模式，使一个类的实例化延迟到其子类。

Factory有工厂的意思，简单来看，这个模式利用到了上一篇的[Template Method模式](https://bugxch.github.io/dp_3/)用来生成具体的实例，说得更清楚一点，工厂方法模式将创建对象的过程延迟到子类实现，其他的父类的步骤保持完整。
## 问题引入
想象一个养殖业的农民，他刚开始仅仅在养马场养马，每个马都需要养殖长大之后在市面出售，后来他获得足够的利润之后，扩展业务也养牛，但是在牛场养牛，每头牛也是需要养殖长大之后在市面出售的。在刚开始的时候，我们需要记录每只马的养殖过程，后面还要记录它的售价。如果你前面只有养马的程序（包括生产、饲养、销售的过程），比如下面这样
```cpp
class Client {
public:
    CreateHorse();
    FeedHorse();
    SellHorse();
private:
    Horse horse_;
}

```
需要添加金养牛的程序，那么大多数情况会出现一个`switch`的分支，随着饲养的品种越来越多，最后会在生产、饲养和售卖的各个过程中出现多个`switch`分支。如果我是农场主，代码会陷入“分支瘫痪”，对维护这一套代码感到厌烦。比如下面这样
```cpp
class Client {
public:
    CreateAnimal() {
        if (animaltype == "horse") {
            // create horse
        } else if (animaltype == "cow") {
            // ceate cow
        } ...
        else {
            // create others
        }
    }
    FeedAnimal() {
        if (animaltype == "horse") {
            // feed horse
        } else if (animaltype == "cow") {
            // feed cow
        } ...
        else {
            // feed others
        }
    }
    SellAnimal() {
        if (animaltype == "horse") {
            // sell horse
        } else if (animaltype == "cow") {
            // sell cow
        } ...
        else {
            // sell others
        }

    }
private:
    int animaltype;
```
这样的方案有什么明显的缺点呢？
- **高耦合**，这个大类中的函数有一处需要添加分支，每个函数就都需要变化，但是每个函数实际是售卖动物的不耦合的步骤（生产不影响饲养，饲养不影响售卖），这些步骤之间耦合太紧，导致“霰弹式修改”；
- **分支瘫痪**，添加的类别越多，代码的`if/else/switch`的分支越多，最后陷入分支瘫痪的状态。
## 解决方案
## UML表示及其代码
## 理解及体会
---

{{< figure src="https://pic.downk.cc/item/5ff098c63ffa7d37b39c239f.jpg" title="欢迎扫码关注我的公众号" >}}
