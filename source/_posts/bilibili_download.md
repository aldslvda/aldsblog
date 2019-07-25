title: 获取BiliBili视频下载地址
date: 2017-02-05 15:22:36
tags:
- Python
- 爬虫
- bilibili	
categories:
- 技术分享	
photos:	
- "https://github.com/aldslvda/blog-images/blob/master/bilibili_banner.png?raw=true"
toc: true
comment: true
---

最近在写Bilibili App的爬虫，由于bilibili是一个视频站，写爬虫很重要的一个环节就是怎样获取到视频的**下载地址**，所以我把如何获取B站视频下载地址做了一个简单的整理。
<!-- more -->
# 获取需要下载视频的av号
一个常见的B站视频的链接如下：

```
http://www.bilibili.com/video/av1608918/
```

拿到av后面的数字，称为**param**

```
param = '1608918'
```

# 获取视频信息
通过抓取B站app发送的HTTP请求，可以知道获取一个视频信息的URL如下：

```
https://app.bilibili.com/x/v2/view?actionKey=appkey&aid=7725070&appkey=27eb53fc9058f8c3&build=4070&device=phone&from=1&mobi_app=iphone&platform=ios&sign=3fa3c3fa6557094b5680066009b41897&ts=1482977603
```

这个URL参数很多，接下来一一作解释。

- actionKey=appkey 
- aid=7725070 param的值 
- appkey=27eb53fc9058f8c3 固定 
- build=4070&device=phone&from=1&mobi_app=iphone&platform=ios 固定 
- sign=3fa3c3fa6557094b5680066009b41897 
- ts=1482977603 时间戳

需要我们其他参数都是现成可以获取到的，而**sign**这个参数是我们需要自己计算得到的，计算方法如下：

## (1) 将除sign外的其他参数按升序排列，得：

```
actionKey=appkey&aid=7725070&appkey=27eb53fc9058f8c3&build=4070&device=phone&from=1&mobi_app=iphone&platform=ios&ts=1482977603
```

## (2) 将步骤1的结果与secretKey进行拼接

   secretKey的取值（根据appKey来获取）： 

```
{"27eb53fc9058f8c3":"c2ed53a74eeefe3cf99fbd01d8c9c375",
"q7R5WBWXdV1T5DO6":"HxibjZZ04WxYn6xVS1q0pIZvxf8b2fOa"}
```

   这里secretKey的值是

```
c2ed53a74eeefe3cf99fbd01d8c9c375
```
   得到结果：

```
actionKey=appkey&aid=7725070&appkey=27eb53fc9058f8c3&build=4070&device=phone&from=1&mobi_app=iphone&platform=ios&ts=1482977603c2ed53a74eeefe3cf99fbd01d8c9c375
```

## (3) 计算步骤2中得到的字符串的md5，得到

```
3fa3c3fa6557094b5680066009b41897
```
这个值就是sign

经过上面的步骤，我们可以得到获取视频信息的URL，通过GET请求进行访问，会返回json格式的视频信息
	
```python
{
    "code":0,
    "data":{
        "aid":7725070,
        "tid":20,
        "tname":"宅舞",
        "copyright":1,
        "pic":"http://i1.hdslb.com/bfs/archive/e0ccafafc5ad49cea65818dcf512858cea00ed88.jpg",
        "title":"【黒kuromi】[963] - [ 梦与叶樱❤ ] - [fgo冲田总司cos",
        "pubdate":1482839192,
        "ctime":1482839192,
        "desc":"自制 翻跳 原振付，音源:sm16595856 dancer: @黒kuromi 摄影感谢:@飘雪幻幻 后期感谢:@怕怕papa ------------------------------------------------- 最近开始玩fgo~身为一个非酋真的很想抽5星呜呜！！ 樱saber太好看ww~~选了冲田总司这套跳了这个比较合适的舞xdd 希望大家看得开心&gt;&lt;~~感谢观看~~",
        "state":0,
        "attribute":1622016,
        "duration":493,
        "tags":[],
        "rights":{
            "bp":0,
            "elec":1,
            "download":1,
            "movie":0,
            "pay":0,
            "hd5":0
        },
        "owner":{
            "mid":427657,
            "name":"黒クロミ",
            "face":"http://i2.hdslb.com/bfs/face/8ec946923ce70e990cf1b05dc3cb14bda6c0d0e0.jpg"
        },
        "stat":{
            "view":170984,
            "danmaku":874,
            "reply":387,
            "favorite":5513,
            "coin":3277,
            "share":183,
            "now_rank":0,
            "his_rank":0
        },
        "pages":[
            {
                "cid":12662538,
                "page":1,
                "from":"vupload",
                "link":"",
                "has_alias":0,
                "weblink":"",
                "part":"",
                "rich_vid":"",
                "vid":"vupload_12662538"
            }
        ],
        "owner_ext":{},
        "req_user":{},
        "online":0,
        "tag":[],
        "relates":[]
    },
    "message":""
}
```

由于过于冗杂，这里只截取了一部分。
其中对于视频下载真正有用的是data下面的pages的值：

```python
{
    "pages":[
        {
            "cid":12662538,
            "page":1,
            "from":"vupload",
            "link":"",
            "has_alias":0,
            "weblink":"",
            "part":"",
            "rich_vid":"",
            "vid":"vupload_12662538"
        }
    ]
}
```

看得出来这是一个**列表**，其实b站的CDN上面，一个较长的视频是由几个较短的视频拼接而成，还有另外一种情况就是b站视频的**分P**，都会使得同一个视频下面存储多个小的视频，pages就是这些小视频的列表，这些视频需要分别获得**下载地址**,我们关注的参数是每一个子视频的**cid**。

# 获取视频的下载地址
下面的链接同样是通过抓取HTTP请求获得：

```
https://interface.bilibili.com/playurl?device=phone&otype=json&buvid=3168fe38e580b16a02c2cc9beceaf6b7&cid=12662538&appkey=YvirImLGlLANCLvM&platform=iphone&build=4070&quality=2&sign=ae1954356fbd510073f636d9ca2d36e7
```

和上一步一样，对请求参数进行分析：

- device=phone 固定 
- otype=json 固定
- buvid=3168fe38e580b16a02c2cc9beceaf6b7 固定 
- cid=12662538 视频的cid 
- appkey=YvirImLGlLANCLvM 计算方法下面会介绍，但是这个参数其实可以直接固定下来。 
- platform=iphone固定 
- build=4070固定 
- quality=2固定
- sign=ae1954356fbd510073f636d9ca2d36e7 这个值同第二步获取视频信息的sign一样需要计算

appKey的计算方式，下面是object-c代码,十分简单易懂，所以直接贴上来

```c
string check_tv_box(string key, int addsum) { 
  	const char* sz = key.c_str(); 
    int len = key.length(); 
    int add = addsum;
    // unsigned char out[17] = {0};
    unsigned char* out = new unsigned char[len + 1]; 
    memset(out, 0, (len + 1) * sizeof(unsigned char)); 
    for (int i = 0; i < len; i++) {
        int index = 0; 
        unsigned char ch = (unsigned char)sz[i]; 
        int num = ch + add; 
        num = (num - 65) % 57 + 65; 
        while (90 < num && 97 > num) {
            add = (index * add) + add; index ++;
            num = ch + add; num = (num - 65) % 57 + 65;
        }
    out[i] = (unsigned char)num;
    }
    string outString = (char*)out; delete [] out; return outString;
}
string sz = "VsfoFjIDZshujsdt"; 
int add = 3; 
string out1 = check_tv_box(sz, add);
```
appKey的初始字符串为"VsfoFjIDZshujsdt”，累加子为3，返回的out1就是加密后的字符串，即appKey的值，这里是YvirImLGlLANCLvM

sign的计算方式也会用到上面的check_tv_box()函数，下面是计算sign的步骤：

## (1) 计算用于拼接的字符串，代码如下：
	
```c
string sz2 = "zEcQEUTunrHlLvYiGXyefkmJPmDQEtow"; 
add = 9; 
string out2 = check_tv_box(sz2, add);
```
	
算法与计算appKey的算法相同，但是初始字符串为"zEcQEUTunrHlLvYiGXyefkmJPmDQEtow”，累加子为9.得到结果：JNlZNgfNGKZEpaDTkCdPQVXntXhuiJEM
	
注意：这个结果固定，可以直接用这个值

## (2) 将其他所有参数排序 

```
appkey=YvirImLGlLANCLvM&build=4070&buvid=3168fe38e580b16a02c2cc9beceaf6b7&cid=12662538&device=phone&otype=json&platform=iphone&quality=2
```

## (3) 在参数最后拼接步骤1的结果： 

```
appkey=YvirImLGlLANCLvM&build=4070&buvid=3168fe38e580b16a02c2cc9beceaf6b7&cid=12662538&device=phone&otype=json&platform=iphone&quality=2JNlZNgfNGKZEpaDTkCdPQVXntXhuiJEM
```

## (4) 计算步骤3结果的md5，得： 

```
ae1954356fbd510073f636d9ca2d36e7
```

计算出了appKey和sign的值，就可以拼接出获取视频下载地址的URL了。
发送请求可以得到返回的json信息：

```python
{
    "from":"local",
    "result":"suee",
    "format":"hdmp4",
    "timelength":493546,
    "accept_format":"mp4,hdmp4,flv",
    "accept_quality":[
        3,
        2,
        1
    ],
    "seek_param":"start",
    "seek_type":"second",
    "durl":[
        {
            "order":1,
            "length":493546,
            "size":72091760,
            "url":" http://ws.acgvideo.com/a/2d/12662538-1-hd.mp4?wsTime=1482992010&wsSecret2=de4372d19062d035396dd2625976ca1a&oi=3664534074&rate=2000"
        }
    ]
}
```
其中durl下面的url参数就是视频下载地址了。


# 写了一个月的第一篇=。=终于写完了
其实工作中碰到过很多app需要抓取信息，但是狠下心来写一篇总结的就只有b站23333

终于是写完了=。=

下一篇依然遥遥无期。。。。

 
