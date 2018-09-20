title: Python常用的一些小技巧(语法糖 etc.)
date: 2018-09-17 14:27:22
tags:
- Python Tricks
- Syntactic sugar
- 语法糖
categories:
- 随手摘录
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/python-tricks-title.jpg?raw=true"
toc: true
comment: true
---


## Python 一些小技巧

本文分享一些使用 Python 的技巧，顺序按照 A-Z 排列。

### all or any

Python 非常受欢迎的原因之一是其可读性和表达性。

人们还经常把 Python 笑称为「可执行伪码（executable pseudocode）」。但是，当你可以编写这样的代码时，很难去反驳这种言论：

```python
x = [True, True, False]

if any(x):

    print("At least one True")

if all(x):

    print("Not one False")

if any(x) and not all(x):

    print("At least one True and one False")
```

### bashplotlib

想在控制台中绘图吗？

```bash
$ pip install bashplotlib
```

使用上面的行，即可在控制台中绘图。

### collections

Python 有一些很棒的默认数据类型，但有时候它们可能不会尽如你意。

不过，Python 标准库提供了 collections 模块。这个方便的附加组件可以为你提供更多数据类型。

> collections 模块：https://docs.python.org/3/library/collections.html

```python
from collections import OrderedDict, Counter

# Remembers the order the keys are added!

x = OrderedDict(a=1, b=2, c=3)

# Counts the frequency of each character

y = Counter("Hello World!")
```

### dir

你是否想过如何查看 Python 对象内部及其具有哪些属性？

输入以下命令行：
```bash
>>> dir()

>>> dir("Hello World")

>>> dir(dir)
```

当以交互方式运行 Python 时，这可能是一个非常有用的功能，并且可以动态地探索你正在使用的对象和模块。

想要了解更多，点这里
> https://docs.python.org/3/library/functions.html#dir


### emoji

是的，真的有。请点击这里
> https://pypi.org/project/emoji/

```bash
$ pip install emoji
```

别以为我不知道你会偷偷试它→→
```python
from emoji import emojize

print(emojize(":thumbs_up:"))

👍
```

### from \_\_future\_\_ import

Python 流行的一个结果是，总有新版本正在开发中。新版本意味着新功能——除非你的版本已经过时。

不过，别担心。\_\_future\_\_模块允许用户导入新版 Python 的功能。这简直就像时间旅行，或者魔法什么的。

> \_\_future\_\_模块：https://docs.python.org/2/library/\*future\*.html

```python
from \_\_future\_\_ import print_function

print("Hello World!")
```

### geopy

地理（Geography）对于程序员来说可能是一个具有挑战性的领域。但是 geopy 模块让它变得异常简单。

> geopy 模块：https://geopy.readthedocs.io/en/latest/
```bash
$ pip install geopy
```

它通过抽取一系列不同地理编码服务的 API 来工作，使用户获取一个地方的完整街道地址、纬度、经度，甚至海拔高度。

另外一个有用的功能是距离：它可以用你喜欢的度量单位计算出两个位置之间的距离。

```python
from geopy import GoogleV3

place = "221b Baker Street, London"

location = GoogleV3().geocode(place)

print(location.address)

print(location.location)
```

### howdoi

陷入编码问题，却不记得以前见过的解决方案？需要检查 StackOverflow，但不想离开终端？

那么你需要这个有用的命令行工具：https://github.com/gleitz/howdoi。

```bash
$ pip install howdoi
```
无论你有什么问题都可以问它，它会尽力回答。
```bash
$ howdoi vertical align css

$ howdoi for loop in java

$ howdoi undo commits in git
```
但是请注意——它会从 StackOverflow 的最高票答案中抓取代码。也就是说它提供的信息并非总是有用……
```bash
$ howdoi exit vim
```
### inspect

Python 的 inspect 模块非常有助于理解问题背后的详情。你甚至可以在 inspect 模块上调用其方法！

> inspect 模块：https://docs.python.org/3/library/inspect.html

下面的代码示例使用 inspect.getsource() 打印自己的源代码。它还使用 inspect.getmodule() 打印定义它的模块。

最后一行代码打印出自己的行号。
```python
import inspect

print(inspect.getsource(inspect.getsource))

print(inspect.getmodule(inspect.getmodule))

print(inspect.currentframe().f_lineno)
```

当然，除了这些琐碎的用途之外，inspect 模块还能帮助你理解代码正在做的事。你还可以用它编写自文档化代码。

### Jedi

Jedi 库是一个自动完成和代码分析的库。它使代码编写变得更快、效果更高。

除非你正在开发自己的 IDE，否则你肯定会对使用 Jedi 库作为编辑插件很感兴趣。

> Jedi：https://jedi.readthedocs.io/en/latest/docs/usage.html

你可能已经在使用 Jedi 了。IPython 项目就使用 Jedi 实现代码自动完成功能。

### \*\*kwargs

学习任何语言时都会遇到很多里程碑。对于 Python 来说，理解神秘的\*\*kwargs 语法可能算是其中之一。

词典对象前面的双星号可以让你把该词典的内容作为命名参数输入到函数中。

词典的秘钥是参数名，值是传递给函数的值。你甚至不需要称它为 kwargs！

```python
dictionary = {"a": 1, "b": 2}

def someFunction(a, b):

    print(a + b)

    return

# these do the same thing:

someFunction(**dictionary)

someFunction(a=1, b=2)
```

当你想编写能够处理事先未定义的命名参数的函数时，这个很有用。

### 列表推导式（List comprehensions）

我最喜欢 Python 编程的原因之一是它的列表推导式
> https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions

这些表达式使得编写干净易读的代码变得很容易，那些代码读起来几乎像自然语言一样。

> 关于它们的更多使用信息请查看：https://www.learnpython.org/en/List_Comprehensions

```python
numbers = [1,2,3,4,5,6,7]

evens = [x for x in numbers if x % 2 is 0]

odds = [y for y in numbers if y not in evens]

cities = ['London', 'Dublin', 'Oslo']

def visit(city):

    print("Welcome to "+city)

for city in cities:

    visit(city)
```
### map

Python 通过许多内置功能支持函数式编程。map() 函数是最有用的函数之一——特别是当它与 lambda 函数结合使用时。

> lambda 函数：https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions

```python
x = [1, 2, 3]

y = map(lambda x : x + 1 , x)

# prints out [2,3,4]

print(list(y))
```

在上面的例子中，map() 将一个简单的 lambda 函数应用于 x 中的每个元素。它返回一个 map 对象，该对象可以被转换成可迭代的对象，如列表或元组。

### newspaper3k

如果你之前没有见过它，那么我建议你先查看
> https://pypi.org/project/newspaper3k/。

它可以帮助你从大量顶级国际出版物中检索到新闻文章和相关元数据。你可以检索图像、文本和作者名。

它还有一些内置的 NLP 功能。

> 地址：https://newspaper.readthedocs.io/en/latest/user_guide/quickstart.html#performing-nlp-on-an-article

如果你想在下一个项目中使用 BeautifulSoup 或其它 DIY 网页抓取库，那么不如使用$ pip install newspaper3k，既省时又省事，何乐而不为呢？

### 运算符重载（Operator overloading）

Python 支持运算符重载。

它实际上是一个简单的概念。你有没有想过为什么 Python 允许用户使用 + 运算符来将数字相加，并级联字符串？这就是运算符重载在发挥作用。

你可以使用 Python 的标准运算符号来定义对象，这样你可以在与这些对象相关的语境中使用它们。
```python
class Thing:

    def __init__(self, value):

        self.__value = value

    def __gt__(self, other):

        return self.__value > other.__value

    def __lt__(self, other):

        return self.__value < other.__value

something = Thing(100)

nothing = Thing(0)

# True

something > nothing

# False

something < nothing

# Error

something + nothing
```

### pprint

Python 的默认 print 函数就可以实现打印功能。但如果尝试打印较大的嵌套对象，就会发现打印结果很丑。

这时 Python 标准库的 pretty printer 模块就可以发挥作用了。该模块可以将复杂的结构化对象以一种易读的格式打印出来。

> pretty printer 模块：https://docs.python.org/3/library/pprint.html

Python 开发者的必备技能之一就是处理复杂的数据结构。
```python
import requests

import pprint

url = 'https://randomuser.me/api/?results=1'

users = requests.get(url).json()

pprint.pprint(users)

Queue
```

### Python 支持多线程，而这是由 Python 标准库的 Queue 模块支持的。

该模块允许用户实现队列（queue）数据结构。队列数据结构允许用户根据特定的规则添加和检索条目。

『First in, first out』 (FIFO) 队列允许用户按照对象被添加的顺序来检索对象。『Last in, first out』 (LIFO) 队列允许用户首先访问最新添加的对象。

最后，优先级队列（priority queue）允许用户根据对象对应的优先级类别来检索对象。

如何使用 queue 在 Python 中实现多线程编程，示例详见：https://www.tutorialspoint.com/python3/python_multithreading.htm。

### \_\_repr\_\_

在 Python 中定义一个类别或对象时，以「官方」方式将对象表示为字符串很有用。例如：
```bash
>>> file = open('file.txt', 'r')

>>> print(file)

<open file 'file.txt', mode 'r' at 0x10d30aaf0>
```
这使代码 debug 变得简单很多。将字符串添加到类别定义，如下所示：

```python
class someClass:

    def __repr__(self):

        return "<some description here>"

someInstance = someClass()

# prints <some description here>

print(someInstance)
```

### sh

Python 是一种伟大的脚本语言，不过有时使用标准 os 和 subprocess 库会有点棘手。

sh 库提供了一种不错的替代方案。

> sh 库：http://amoffat.github.io/sh/

该库允许用户像使用普通函数一样调用任意程序，这对自动化工作流和任务非常有用。
```python
from sh import *

sh.pwd()

sh.mkdir('new_folder')

sh.touch('new_file.txt')

sh.whoami()

sh.echo('This is great!')
```

### 类型提示（Type hints）

Python 是动态语言。在定义变量、函数、类别等时无需指定数据类型。

这有利于缩短开发周期。但是，简单的类型错误（typing issue）导致的运行时错误真的太烦了。

从 Python 3.5 版本开始，用户可以选择在定义函数时开启类型提示。
```python
def addTwo(x : Int) -> Int:

    return x + 2
```
你还可以定义类型别名：
```python
from typing import List

Vector = List[float]

Matrix = List[Vector]

def addMatrix(a : Matrix, b : Matrix) -> Matrix:

  result = []

  for i,row in enumerate(a):

    result_row =[]

    for j, col in enumerate(row):

      result_row += [a[i][j] + b[i][j]]

    result += [result_row]

  return result

x = [[1.0, 0.0], [0.0, 1.0]]

y = [[2.0, 1.0], [0.0, -2.0]]

z = addMatrix(x, y)
```
尽管非强制，但类型注释可以使代码更易理解。

它们还允许你在运行之前使用类型检查工具捕捉 TypeError。在进行大型复杂项目时执行此类操作是值得的。

### uuid

生成通用唯一标识符（Universally Unique ID，UUID）的一种快速简单方法就是使用 Python 标准库的 uuid 模块。

> uuid 模块：https://docs.python.org/3/library/uuid.html

```python
import uuid

user_id = uuid.uuid4()

print(user_id)
```

这创建了一个随机化后的 128 比特数字，该数字几乎必然是唯一的。

事实上，可以生成 2¹²²可能的 UUID。这个数字超过了 5,000,000,000,000,000,000,000,000,000,000,000,000。

在给定集合中找出重复数字的可能性极低。即使有一万亿 UUID，重复数字存在的概率也远远低于十亿分之一。

### 虚拟环境（Virtual environment）

这可能是 Python 中我最喜欢的事物了。

你可能同时处理多个 Python 项目。不幸的是，有时候两个项目依赖于相同依赖项的不同版本。那你要安装哪个版本呢？

幸运的是，Python 支持虚拟环境，这使得用户能够充分利用两种环境。见下列行：
```bash
python -m venv my-project

source my-project/bin/activate

pip install all-the-modules 
```
现在你在一台机器上具备独立的多个 Python 版本了。问题解决！

### wikipedia

Wikipedia 拥有一个很棒的 API，允许用户以编程方式访问巨大体量的免费知识和信息。

wikipedia 模块使得访问该 API 非常便捷。

> Wikipedia 模块：https://wikipedia.readthedocs.io/en/latest/quickstart.html

```python
import wikipedia

result = wikipedia.page('freeCodeCamp')

print(result.summary)

for link in result.links:

    print(link)
```
和真实的维基百科网站类似，该模块支持多种语言、页面消歧、随机页面检索，甚至还具备 donate() 方法。

### xkcd

humour 是 Python 语言的一个关键特征，其名称来自英国喜剧片《蒙提·派森的飞行马戏团》(Monty Python and the Flying Circus)。Python 的很多官方文档引用了该喜剧片最著名的剧情。

幽默感并不限于文档。试着运行下列行：
```python
import antigravity
```
将打开 xkcd 画的 Python 漫画。不要改变这一点，Python。不要改变。

### YAML

YAML 代表 『YAML Ain』t Markup Language』。它是一种数据格式语言，是 JSON 的超集。

与 JSON 不同，它可以存储更复杂的对象并引用自己的元素。你还可以编写注释，使其尤其适用于编写配置文件。

> PyYAML 模块（https://pyyaml.org/wiki/PyYAMLDocumentation）可以让你在 Python 中使用 YAML。

安装：
```basj
$ pip install pyyaml
```
然后导入到项目中：
```python
import yaml
```
PyYAML 使你能够存储任何数据类型的 Python 对象，以及任何用户定义类别的实例。

### zip

给你支最后一招，非常酷。还在用两个列表来组成一部词典吗？
```python
keys = ['a', 'b', 'c']

vals = [1, 2, 3]

zipped = dict(zip(keys, vals))
```
zip() 内置函数使用多个可迭代对象作为输入并返回元组列表。每个元组按位置索引对输入对象的元素进行分组。

你也可以通过调用\*zip() 来「解压」对象。


原文链接：https://medium.freecodecamp.org/an-a-z-of-useful-python-tricks-b467524ee747