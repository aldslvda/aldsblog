title: fluent python 第十六章 协程
date: 2020-06-10 17:17:00
tags:
- Python
- coroutines
- 协程
  
categories:
- 读书笔记
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/fluent_python_logo.png?raw=true"
toc: true
comment: true
---

> Fluent Python 第十六章读书笔记

<!-- more -->

# Chapter 16. Coroutines
# 第十六章: 协程

协程与生成器类似，都是定义体中包含 yield 关键字的函数。可是，在协 程中，yield 通常出现在表达式的右边（例如，datum = yield），可以产出值，也可 以不产出——如果 yield 关键字后面没有表达式，那么生成器产出 None。协程可能会从 调用方接收数据，不过调用方把数据提供给协程使用的是 .send(datum) 方法，而不是 next(...) 函数。通常，调用方会把值推送给协程。

yield 关键字甚至还可以不接收或传出数据。不管数据如何流动，**yield 都是一种流程控制工具**，使用它可以实现协作式多任务：协程可以把控制器让步给中心调度程序，从而激 活其他的协程。

- 生成器作为协程使用时的行为和状态
- 使用装饰器自动预激协程
- 调用方如何使用生成器对象的 .close() 和 .throw(...) 方法控制协程
- 协程终止时如何返回值
- yield from 新句法的用途和语义

## 16.1 生成器进化成协程

PEP 342 中给生成器API中增加了.send(...)， .throw(...) 和 .close() 方法， 生成器的调用方可以使用.send(...) 方法发送数据，发送的数据会成为生成器函数中 yield 表达式的值。使用这种办法，生成器可以作为协程使用。协程是一种和调用方合作的过程，可以接受和产出由调用方提供的数据。.throw(...) 让调用方抛出异常，.close()终止协程。

# 16.2 用作协程的生成器的基本行为
先上代码:

```python
>>> def simple_coroutine(): # ➊ 
...     print('-> coroutine started') 
...     x = yield # ➋ 
...     print('-> coroutine received:', x) ...

>>> my_coro = simple_coroutine()
>>> my_coro # ➌ 
<generator object simple_coroutine at 0x100c2be10>
>>> next(my_coro) # ➍
-> coroutine started
>>> my_coro.send(42) # ➎
-> coroutine received: 42 Traceback (most recent call last): # ➏ 
...
StopIteration
```

