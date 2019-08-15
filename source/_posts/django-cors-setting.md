title: Django 解决跨域问题
date: 2019-08-15 16:39:18
tags:
- Django
- CORS
categories:
- 疑难问题
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/CORS.jpeg?raw=true"
toc: true
comment: true
---

##1. 使用django-cors-headers解决

### 1.1 安装django-cors-headers

```$ pip install django-cors-headers```


### 1.2 修改settings.py
```python
INSTALLED_APPS = [
    ......
    'corsheaders',
    ......
]

MIDDLEWARE = [
    ......
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    ......
]

CORS_ORIGIN_ALLOW_ALL = True
CORS_ALLOW_CREDENTIALS = True
```


## 2. 使用自定义middleware

```python
class EnableCors(MiddlewareMixin):
    @staticmethod
    def process_response(request, response):
        response["Access-Control-Allow-Origin"] = "*"
        # print(response["Access-Control-Allow-Origin"])
        return response
```

## 3. 我遇到的问题
以前都是直接用1就解决了，没想到今天吃瘪了，懵逼了好一会。。
上面两种方式都试过了，前端始终报错，于是用```httpie```测试了一下接口
结果如下

```
-> % http -h GET http://xxxx/
HTTP/1.1 302 Moved Temporarily
Connection: keep-alive
Content-Length: 169
Content-Type: text/html
Date: Thu, 15 Aug 2019 08:34:48 GMT
Location: https://xxxx/
Server: stgw/1.3.10.9_1.13.5
```
破案了，因为公司运维给配的域名自动会把http请求重定向到https，导致返回头没有```Access-Control-Allow-Origin```字段，也使得跨域的设置失效。