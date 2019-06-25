title: RESTful风格简介
date: 2019-06-25 17:09:00
tags:
- RESTful
categories:
- 学习记录	
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/rest_logo.png?raw=true"
toc: true
comment: true
---


### RESTful 风格简介

#### 什么是REST   
REST, **RE**presentational **S**tate **T**ransfer, 表现层状态转移是一种编程风格。它描述了两个系统(典型的是客户端与服务端)通过网络交互的一系列准则。   
REST 是一种风格，是指导方针，而不是规定的细则或者标准。所以只是有“最佳实践”, 而没有“规范说明”。最早由Roy Fielding于2000年在他的博士论文"Architectural Styles and the Design of Network-based Software Architectures"中提出。  
REST是基于**资源**的，下面是REST风格的一部分：

> Things vs actions
> Nouns vs verbs
> Resources identified by URIs . Multiple URIs may refer to the same resource
> Resources are separate from their representation

##### representational 表现层   
表现层是指资源的表现形式，也就是资源的信息是如何组织的。他能部分代表资源的状态，表现层在客户端与服务端之间交互，典型的形式是XML/JSON
举个例子，数据库中存有一个人的个人信息，那么:

- 资源: 这个人的个人信息
- 服务: 获取联系方式 (GET)
- 表现层:
    人名，住址，电话(JSON/XML)

#### 为什么使用RESTful API 
- 设计简洁
- 遵循标准的HTTP协议
- 独立于编程语言之外
- 最重要的是: 服务端与客户端的轻耦合，它让服务端对客户端"不可知"，只要能使用HTTP协议进行交互，不论是浏览器，手机还是其他设备都无所谓。

#### RESTful 6个指导性的约束
- Uniform Interface 统一的接口
- Stateless 无状态
- Cacheable 可缓存
- server-client 服务端-客户端
- 分层的系统
- 按需代码

##### Uniform Interface 统一的接口
资源的定位: 独立的资源会被HTTP请求定位, 比方说使用 URI. 在概念上，资源是与表现层分离的，客户端请求的得到的数据是资源的表现层而不是资源本身。

通过表现层操作资源: 当客户端拥有了(附加了一些元信息，如HTTP HEADERs的)资源的表现层, 它就有了足够的权限对资源进行修改或者删除。

自描述信息: 每个HTTP消息包含足够的信息，告诉接收端怎样处理这个信息。

HATEOAS(Hypermedia as the Engine of Application State, 超媒体即应用状态引擎):  当客户端游泳一个初始3的URI，REST应用应该可以动态利用服务端提供的链接来访问到所有资源或者进行所有操作。

##### Stateless 无状态
**无状态原则**
Statelessness：无状态原则是RESTful架构设计中一个非常重要的原则，无状态是相对于有状态而言的。在理解什么是无状态的交互请求之前，首先我们需要了解什么是有状态，并对两者进行比较以加深认识。

**Web服务的状态**
Web服务建立在Web应用程序的协议之上，如：HTTP协议。Web服务的状态指的是请求的状态，而不是资源的状态。是两个关联用户(Client与Server)进行交互操作时所留下来的公共信息(工作流、用户状态信息等数据)。这些信息可以被指定在不同的作用域中，如：page、request、session、application或全局作用域，一般由Server中的Session来保存这些信息。

**有状态的Web服务**
在基于状态的Web服务中，Client与Server交互的信息(如：用户登录状态)会保存在Server的Session中。再这样的前提下，Client中的用户请求只能被保存有此用户相关状态信息的服务器所接受和理解，这也就意味着在基于状态的Web系统中的Server无法对用户请求进行负载均衡等自由的调度(一个Client请求只能由一个指定的Server处理)。同时这也会导致另外一个容错性的问题，如果指定的Server在Client的用户发出请求的过程中宕机，那么此用户最近的所有交互操作将无法被转移至别的Server上，即此请求将无效化。

**无状态的Web服务**
在无状态的Web服务中，每一个Web请求都必须是独立的，请求之间是完全分离的。Server没有保存Client的状态信息，所以Client发送的请求必须包含有能够让服务器理解请求的全部信息，包括自己的状态信息。使得一个Client的Web请求能够被任何可用的Server应答，从而将Web系统扩展到大量的Client中。

**两者的区别**
因为无状态原则的特性，让RESTful在分布式系统中得到了广泛的应用，它改善了分布式系统的可见性、可靠性以及可伸缩性，同时有效的降低了Client与Server之间的交互延迟。无状态的请求有利于实现负载均衡，在分布式web系统下，有多个可的Server，每个Server都可以处理Client发送的请求。有状态的请求的状态信息只保存在第一次接收请求的Server上，所以后来同一个Client的请求都只能由这台Server来处理，Server无法自由调度请求。无状态请求则完全没有这个限制。其次，无状态请求有较强的容错性和可伸缩性。如果一台服务器宕机，无状态请求可以透明地交由另一台可用Server来处理，而有状态的请求则会因为存储请求状态信息的Server宕机而承担状态丢失的风险。Restful风格的无状态约束要求Server不保存请求状态，如果确实需要维持用户状态，也应由Client负责。例如：  

> 使用Cookies通过客户端保持登陆状态：

> 在REST中，每一个对象都是通过URL来表示，对象用户负责将状态信息打包进每一条信息内，保证对象的处理总是无状态的。在HTTP服务器中，服务器没有保存客户端的状态信息，客户端必须每次都带上自己的状态去请求服务器。客户端以URL形式提交的请求包含了cookies等带状态的数据，这些数据完全指定了所需的登录信息，而不需要其他请求的上下文或内存。

> 传递User credentials是无状态的，而传递SessionID是有状态的,因为session信息保存在服务器端。

> **无状态请求**：Server不保存任何请求状态信息，Client的每一个请求都具有User credentials等所需要的全部信息，所以能被任意可用的Server应答。   

> **有状态请求**：Server保存了Client的请求状态，Server会通过Client传递的SessionID在Server中的Session作用域找到之前交互的信息，并以此来实现应答。所以Client只能由某一个Server来应答。


##### Cacheable 可缓存
互联网中的客户端和中间层服务器可以缓存响应。  
因此响应必须直接或间接定义自身是否可被缓存，以免客户端使用过期的响应数据来发送其它请求。  
良好的缓存策略可以有效减少客户端-服务器之间的交互，从而进一步提高系统的可伸缩性和性能。  

##### Server-Client 服务端-客户端

> wiki: [客户端-服务端模型](https://en.wikipedia.org/wiki/Client%E2%80%93server_model)

客户端-服务端分离:   
通过分离用户界面和数据存储这两个关注点，提高了用户界面跨平台的可能性，通过简化服务器组件提高了其可伸缩性。  
这种分离对于Web来说更加重要的意义是，它使得组件能够进行独立修改和扩展，从而能够支持大量组织的网络化需求。   

##### Layered system 分层系统

客户端通常无法判断它是否是直接连接到后端服务器，还是中间服务器。   
中间服务器可通过启用负载平衡，并通过提供共享高速缓存来提高系统的可扩展性。   
当然也可以强制执行安全政策。  

##### Code on demand 按需代码（可选）

服务端可以通过传递可执行代码临时为客户端扩展或自定义功能。  
这方面的例子包括可编译的组件如Java applets和客户机端脚本如JavaScript。



