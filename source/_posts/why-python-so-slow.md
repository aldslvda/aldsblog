title: 为什么Python 如此之慢
date: 2019-05-30 10:37:00
tags:
- Python
- GIL
- 动态类型语言
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/python_slow.png?raw=true"
categories:
- 随手摘录
toc: true
comment: true
---

## 为什么Python 如此之慢

原文链接: https://hackernoon.com/why-is-python-so-slow-e5074b6fe55b

Python 正在爆炸般流行起来，它被用于DevOps, 数据处理，web开发和安全领域。但是在速度方面却没有取得过什么胜利。    
Java在速度方面和C/C++/C#/Python比起来如何？答案很大程度上取决于你所运行的应用。没有什么跑分是完美的，但是编程语言测评游戏(Computer Language Benchmarks Game)是一个很好的切入点。  

十多年来编程语言测评游戏一直对我来说是一个参考，相比于其他语言，比如Java, C#, Go, JavaScript, C++, Python是最慢的语言之一。这些语言中包含了JIT编译器(c#, java)，AOT编译器(c/c++)和解释型语言js。

注意: 当我说“Python”, 我说的是Python的参考实现CPython, 其他实现这篇文章也会提到。
这篇文章想要回答的一个问题就是：当Python执行一个相同的程序要比别的语言实现慢2-10倍的时候，为什么会这么慢，难道我们不能让它跑的更快吗？

下面说一下本文的主要观点(当然也是客观事实):
"因为Python 有GIL(Global Interpreter Lock, 全局解释锁)"
“因为Python 是解释型语言而不是编译型”
“因为Python 是动态类型语言”
哪一个原因影响最大对速度影响最大呢？

### GIL

现代计算机的cpu有多核，有时候有多个处理器。为了充分利用多出来的处理能力，操作系统定义了一种更加低级别的单位:线程，让一个进程可以启动多个线程来执行系统指令。
这样的话如果一个进程的CPU资源非常紧张，负载就会均匀的平摊到各个CPU核心，这种方法能够很高效地让大多数任务更快完成。
当我写这篇文章的时候，我的Chrome浏览器开启了44个线程。要记住的是线程的结构和api在POSIX-based的系统(OSX, Linux)和Windows下面是不一样的。操作系统同样会管理线程的调度。
如果你没有进行过多线程编程，你需要快速熟悉一下"锁"的概念。不像单线程的进程，多线程的进程中，当你更改内存中的变量，你需要确认多个线程不会同时尝试访问或修改同一个内存块。

当CPython创建变量，它会给变量分配内存空间并且计算变量的引用数，如果引用数为0, Python将会释放掉这块内存，这就是为什么for循环表达式里的临时变量不会让内存爆炸。也是CPython的垃圾回收机制。

接下来的挑战是，当变量被多个线程共享时，如何锁住引用数。Python程序执行过程中有一个全局解释锁GIL 小心地控制着线程的执行。Python解释器在同一时间只能执行一个操作，不论有多少个线程。

### 这对于Python应用的性能表现意味着什么呢？
如果你的应用是单线程单解释器的，这对你的应用性能毫无影响。移除GIL也不会对你的应用性能有任何影响。
如果你想在同一个解释器中使用线程进行并发操作，并且你的线程是IO密集型的，那你就会看到GIL的资源争夺。

David Beazley的博客中对多线程程序中 GIL 的作用进行了可视化:http://dabeaz.blogspot.com/2010/01/python-gil-visualized.html

下图表示了Python多线程程序中GIL的分配情况。

![1](https://github.com/aldslvda/blog-images/blob/master/Thread-Excution-Model.jpg?raw=true)

如果一个Python程序里有多个线程，一个程序运行的时候会拿着GIL，当遇到I/O的时候会放开GIL，但是CPU-bound的线程通常不会进行I/O。Python切换线程的一种作法是每100 ticks检查一下，可以通过sys.setcheckinterval()修改这个数值。
综上所述，因为Python线程不能有效利用多核，但是增加了CPU context switch的消耗，所以对于CPU-bound的程序表现很差。更糟糕的是，在多核情况下可能表现会更差，因为系统支持多线程运行但是GIL保证只有一个线程运行，这时候多线程会反复的检查GIL是否被释放，但是拿不到GIL(因为有太多线程竞争)，有可能导致系统发生Trashing现象。

如果你使用一个web应用(Django)并且使用WSGI，每个请求会有一个单独的Python解释器，由于Python的全局解释锁启动很慢，所以有的WSGI实现会有一个“守护模式”，就是先把Python进程启动起来放着，等待请求进来使用。

### 那其他的Python实现呢？
PyPy也有GIL但是比CPython快超过三倍。
JPython 没有GIL, 因为JPython的线程代表一个Java线程，得益于JVM的内存管理机制，JPython不使用GIL。

### JavaScript是怎么做的呢
首先所有的Javascript 引擎使用标记-清除的垃圾回收机制。 上面提到过， GIL的需求主要是因为CPython的内存管理算法。
JavaScript 没有GIL, 但它同时也是单线程的所以它并不需要GIL。JavaScript使用事件循环和Promise/Callback实现异步的编程而不是使用并发。Python也有类似实现asyncio

### “因为Python是解释型语言”

我经常听说这个言论，我觉得这是一个对于CPython执行方式的粗暴简化。
如果你在终端执行一个命令(比如python myscript.py)，CPython会开始顺序执行一大串任务: 读取，词法分析，解析，编译，解释，执行代码。

一个重点是.pyc文件的创建。在编译阶段，字节码串被写到__pycache__/下(3.x)或者和py文件相同的文件夹(2.x)。这个操作不仅对你自己的代码有效，也包括所有你导入的模块。

所以大多数情况下，Python在本地解释和执行字节码，与之相比 Java 和 C#.NET:    
Java 编译成一个“中间语言”，然后JVM读字节码实时将其转化为机器码，.NET的CIL也一样，.NET CLR(Common-Language-Runtime, 通用语言运行时)，使用的是实时编译到机器码(JIT)。

所以，如果它们都用到了虚拟机和部分字节码，为什么Python在跑分上比Java/C#慢这么多呢?
Java/C# 是即时编译(JIT)的。   
JIT要求一个中间语言，以便让代码转换成区块。AOT编译是为了可以在交互前保证CPU能理解每一行代码。  
JIT本身不会让执行速度更快，以为它仍然是在执行相同的字节码， 然而JIT可以让实时优化成为可能，一个好的JIT编译器应用的那一部分被执行很多次，这些部分被称为“热点”。
这样编译器会将这些部分替换成更加高效的版本。这意味着如果你的程序重复做相同的事情，使用JIT就能显著提高速度。同时Java/C#是强类型的语言所以优化器可以对语言作出更多预设。

PyPy 使用JIT，前面提到，它比CPython快得多。
### 所以为什么CPython不用JIT呢

JIT有很多缺点，其中一个就是启动慢。CPython 启动已经相对很慢了, PyPy比CPython启动慢CPython2-3倍。JVM启动是出了名的慢。.NET CLR使用随操作系统启动来解决这个问题, 但CLR的开发者同时也是其依赖的系统的开发者(win)。
如果你的Python是单进程，运行时间很长，并且有很多重复操作可以被优化，那么使用JIT就很有意义。
但是CPython是通用实现，当使用Python开发命令行工具，每次都等待JIT启动是非常糟糕的体验。
CPython需要服务于尽可能广泛的场景，有可能使用JIT反而会大幅度拖累系统性能。
如果你需要使用JIT并且有一个适合的工作场景，那可以使用PyPy。

### “因为Python是动态类型语言”

在静态类型语言中，声明变量之前需要声明变量类型，包括 C, C++, Java, C#, Go。动态类型语言中仍然有类型的概念，但是变量的类型可变。

```python
a = 1
a = "foo"
```

这个小例子中, Python 使用相同的变量名创建了类型不同的第二个变量，释放掉了第一个变量的内存空间。   
静态类型语言并不是为了麻烦你而设计的，而是根据CPU行为设计的。如果所有的行为最终会变成二进制操作，你必须将对象转换成更加底层的数据结构。

Python为你代劳了转换到底层数据结构这一步，所以你不用关心。当然不用声明变量并不是Python慢的原因。
Python语言的设计极为灵活，几乎所有东西都能变成动态的，比如猴子补丁。这种设计让Python的优化变得难以想象的困难。
> 猴子补丁:
> 1. 在运行时替换方法、属性等
> 2. 在不修改第三方代码的情况下增加原来不支持的功能
> 3. 在运行时为内存中的对象增加patch而不是在磁盘的源代码中增加

python中一个很简单的例子：

```python
import simplejson as json
```

### 所以是Python的动态性让它如此慢吗？
比较和转换类型非常耗费资源, 每次对变量读写、引用都需要检查类型。如此动态化的语言是很难优化的，很多Python的不同实现能更快是因为为了性能在灵活性方面做了妥协。

比如CPython, 将C语言的静态类型和Python结合，这些静态类型能提供84倍的性能优化。
### 结论

Python这么慢主要是因为动态的生态和它的多功能性。它能用来解决所有类型的问题，所以在不同领域我们可以选择更加快速的Python版本。  
有很多方法可以用来优化你的Python程序，比如活用async, 了解分析工具, 考虑使用多个解释器等等。比如一些启动时间不重要的应用，或者能够能JIT获益的程序我们可以使用PyPy。如果你的代码有些部分非常要求性能，又使用了很多C语言的静态类型，那么选择Cython。
