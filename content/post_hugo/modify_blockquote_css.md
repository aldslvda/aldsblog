+++
tags = ["博客","CSS"]
topics = ["CSS"
]
description = "修改博客的引用块(blockquote)样式"
date = "2017-12-17T11:29:51+08:00"
title = "修改博客的引用块(blockquote)样式"
draft = false

+++

  
最近写博客的时候发现引用的样式特别不明显![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/05.png?raw=true),emmmm 就像这样:
![修改前的样式](https://github.com/aldslvda/blog-images/blob/master/before_modify_blockquote.png?raw=true)

用浏览器工具查看了一下CSS样式, 是浏览器自带的样式（基本只有个缩进

```css   
/*user agent stylesheet*/
blockquote {
    display: block;
    -webkit-margin-before: 1em;
    -webkit-margin-after: 1em;
    -webkit-margin-start: 40px;
    -webkit-margin-end: 40px;
}
```

接下来去查了一下,这个标签也没啥特别的，和别的标签的css设置没啥不一样，就自己做了一个比较符合blackburn主题（又不是太丑 的样式

```css   
/*customized stylesheet*/
blockquote{
    background: #eee; /*背景颜色*/
    padding: 1px 1px 1px 5px; /*引用块内部文字距离边框的距离*/
    margin: 1em 3em 1em 2em; /*引用块边框上右下左的缩进*/
    border-style:solid; /*边框样式*/
    border-width:0px 0px 0px 7px;/*边框宽度*/
    border-color:#666;  /*边框颜色*/
  }
```
修改后是这样:     
![修改后的样式](https://github.com/aldslvda/blog-images/blob/master/after_modify_blockquote.png?raw=true)

还...还行吧![](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/11.png?raw=true)

差不多这样就结束了（这个可比读书笔记轻松多了![](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/13.png?raw=true)
