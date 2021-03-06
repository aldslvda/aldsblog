title: Fluent Python 第十一章 从协议到抽象基类
date: 2018-07-28 18:00:00
tags:
- Python
- duck typing
- protocal
- abstract basic class
- ABC
- fluent python
- interfaces
categories:
- 读书笔记
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/fluent_python_logo.png?raw=true"
toc: true
comment: true
---

> Fluent Python 第十二章读书报告

<!-- more -->

# Chapter 11. Interfaces From Protocols to ABCs
# 第十一章: 从协议到抽象基类

本章主要讨论接口，从鸭子类型的动态协议到使接口更加明确，能验证实现是否符合规定的抽象基类（ABC）

本章会专门讲解抽象基类。首先，本章说明抽象基类的常见用途：实现接口时作为超类使用。然后，说明抽象基类如何检查具体子类是否符合接口定义，以及如何使用注册机制声明一个类实现了某个接口，而不进行子类化操作。最后，说明如何让抽象基类自动“识别”任何符合接口的类——不进行子类化或注册。

## Python 中的接口和协议

Python中，我们把协议定为非正式的接口， 协议也是Python这类动态语言实现多态的方式。

Python 中接口的运作方式: Python 中没有interface 关键字， 并且除了抽象基类（ABC）,每个类都有接口，实现方式为：类实现或继承公开属性(方法或者数据属性)

关于接口有一个实用的补充定义：对象公开方法的子集，让对象在系统中扮演特定角色。
借口是实现特定角色的方法的集合，这就是协议。协议与继承没有关系，一个类可能实现多个接口使同一个实例扮演多个角色。

协议不是正式的接口, 没有接口一致性的各种强制，因此一个类可以只实现部分接口。


## Python中的序列协议

Python中数据模型的哲学是尽量支持基本协议， 下面的图展示了抽象基类Sequence的正式接口。

![Figure-11-1](https://github.com/aldslvda/blog-images/blob/master/fluent-python-11.1.png?raw=true)

如果没有实现\_\_iter\_\_和\_\_contains\_\_方法， Python会调用\_\_getitem\_\_方法， 设法让迭代和in运算符可用。几十一个对象只实现了\_\_getitem\_\_方法，也能进行迭代，为了迭代对象，解释器会尝试调用两个不同的方法。


## 使用猴子补丁在运行时实现协议

> 得益于鸭子类型，如果遵守既定的协议，很有可能增加利用现有的标准库和第三方代码的可能性。
> 猴子补丁: 在运行时修改类或者模块，而不改动源码。这种技术非常强大，但是打补丁的代码和被打补丁的程序需要耦合非常紧密，而且往往要处理没有文档的部分。



协议可以支持猴子补丁， 恰恰说明了协议的动态性： 即使对象一开始没有实现所需的方法，后来用补丁的形式加进去也行。这也是鸭子类型思想的一个缩影。


## 关于抽象基类和白鹅类型

> 白鹅类型： 只要cls是抽象基类， 即cls的元类是abc.ABCMeta, 就可以使用isinstance(obj, cls)
- 抽象基类的本质就是几个特殊方法的集合
- 可以用instance(obj, cls)检查类是否已经实现了抽象基类定义的api契约。
- 需要注意的是 生产代码中尽量避免定义抽象基类，极容易因为设计不当造成滥用，滥用抽象基类会造成**灾难性的后果**
- 抽象基类是封装框架引入一般性概念和抽象的。


## 定义抽象基类的子类

要想实现子类，可以覆盖从抽象基类中继承的方法，以更高效的方式重新实现。例如\_\_contains\_\_方法会扫描序列，如果你定义的序列按顺序排列，那么就可以重新定义这个方法使用bisect函数二分查找。

## 标准库中的抽象基类

### collections.abc模块
Python 标准库中有两个abc模块，一个是collections.abc， 另一个是abc.ABC，后者是所有抽象基类的依赖。
collections.abc类中有16个抽象基类，它们的继承关系如下图所示:

![Figure-11-2](https://github.com/aldslvda/blog-images/blob/master/fluent-python-11.2.png?raw=true)

- Iterable、Container 和 Sized
    各个集合应该继承这三个抽象基类，或者至少实现兼容的协议。Iterable 通过\_\_iter\_\_方法支持迭代，Container 通过 \_\_contains\_\_ 方法支持 in 运算符，Sized通过 \_\_len\_\_ 方法支持 len() 函数。

- Sequence、Mapping 和 Set
　　这三个是主要的不可变集合类型，而且各自都有可变的子类。

- MappingView
　　在 Python 3 中，映射方法 .items()、.keys() 和 .values() 返回的对象分别是ItemsView 、KeysView 和 ValuesView 的实例。前两个类还从 Set 类继承了丰富的接口，包含 3.8.3 节所述的全部运算符。

- Callable 和 Hashable
　　这两个抽象基类与集合没有太大的关系，只不过因为 collections.abc 是标准库中定义抽象基类的第一个模块，而它们又太重要了，因此才把它们放到 collections.abc模块中。这两个抽象基类的主要作用是为内置函数 isinstance 提供支持，以一种安全的方式判断对象能不能调用或散列。若想检查是否能调用，可以使用内置的 callable() 函数；但是没有类似的 hashable() 函数，因此测试对象是否可散列，最好使用 isinstance(my_obj, Hashable)。

- Iterator
　　它是 Iterable 的子类。

### 抽象基类中的数字塔numbers

numbers包定义的是数字塔，包含的类如下，通过类名即可判断出他们的继承关系:

- Number
- Complex
- Real
- Rational
- Integral

# 本章小结

本章介绍了Python中非正式接口(协议的高动态本性)，以及Python中接口的运作方式，同时介绍了Python中的抽象基类及其类别和用途。

> 尽管抽象基类使得类型检查变得更容易了，但不应该在程序中过度使用它。Python 的核心在于它是一门动态语言，它带来了极大的灵活性。如果处处都强制实行类型约束，那么会使代码变得更加复杂，而本不应该如此。我们应该拥抱 Python 的灵活性.
> "如果觉得自己想创建新的抽象基类，先试着通过常规的鸭子类型来解决问题。”



