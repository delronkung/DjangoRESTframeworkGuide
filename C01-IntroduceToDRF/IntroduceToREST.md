# 认识RESTful

**在前后端分离的应用模式里，后端API接口如何定义？**

例如对于后端数据库中保存了商品的信息，前端可能需要对商品数据进行增删改查，那相应的每个操作后端都需要提供一个API接口：

1. POST   /add-goods    增加商品
2. POST   /delete-goods  删除商品
3. POST   /update-goods   修改商品
4. GET   /get-goods   查询商品信息

对于接口的请求方式与路径，每个后端开发人员可能都有自己的定义方式，风格迥异。

是否存在一种统一的定义方式，被广大开发人员接受认可的方式呢？

这就是被普遍采用的API的RESTful设计风格。

## 1.  起源

REST这个词，是[Roy Thomas Fielding](http://en.wikipedia.org/wiki/Roy_Fielding)在他2000年的[博士论文](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)中提出的。

![REST作者](/images/REST作者.jpg)

Fielding是一个非常重要的人，他是HTTP协议（1.0版和1.1版）的主要设计者、Apache服务器软件的作者之一、Apache基金会的第一任主席。所以，他的这篇论文一经发表，就引起了关注，并且立即对互联网开发产生了深远的影响。

## 2.  名称

Fielding将他对互联网软件的架构原则，定名为**REST**，即**Representational State Transfer**的缩写。维基百科称其为“**具象状态传输**”，国内大部分人理解为“**表现层状态转化**”。

**RESTful是一种开发理念**。维基百科说：**REST是设计风格而不是标准**。 REST描述的是在网络中client和server的一种交互形式；REST本身不实用，实用的是如何设计 RESTful API（REST风格的网络接口）,一种万维网软件架构风格。

我们先来具体看下RESTful风格的url,比如我要查询商品信息，那么

> 非REST的url：**http://.../queryGoods?id=1001&type=t01**
>
> REST的url: **http://.../t01/goods/1001**

可以看出**REST特点：url简洁，将参数通过url传到服务器，**而传统的url比较啰嗦，而且现实中浏览器地址栏会拼接一大串字符，想必你们都见过吧。但是采用REST的风格就会好很多，现在很多的网站已经采用这种风格了，这也是潮流方向，典型的就是url的短化转换。

**那么，到底什么是RESTFul架构： 如果一个架构符合REST原则，就称它为RESTful架构。**

要理解RESTful架构，理解Representational State Transfer这三个单词的意思。

- **具象的**，就是指表现层，要表现的对象也就是“资源”，什么是资源呢？网站就是资源共享的东西，客户端（浏览器）访问web服务器，所获取的就叫资源。比如html，txt，json，图片，视频等等。

- **表现**，比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。

  浏览器通过URL确定一个资源，但是如何确定它的具体表现形式呢？应该在HTTP请求的头信息中用Accept和Content-Type字段指定，这两个字段才是对"表现层"的描述。

- **状态转换，** 就是客户端和服务器互动的一个过程，在这个过程中, 势必涉及到数据和状态的变化, 这种变化叫做状态转换。

  互联网通信协议HTTP协议，客户端访问必然使用HTTP协议**，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。**

  HTTP协议实际上含有4个表示操作方式的动词，分别是 GET,POST,PUT,DELETE,他们分别对应四种操作。GET用于获取资源，POST用于新建资源，PUT用于更新资源，DElETE用于删除资源。GET和POST是表单提交的两种基本方式，比较常见，而PUT和DElETE不太常用。

  而且HTTP协议是一种无状态协议，这样就必须把所有的状态都保存在服务器端**。**因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）

## 3.  总结

**综合上面的解释，RESTful架构就是：**
- **每一个URL代表一种资源；**
- **客户端和服务器之间，传递这种资源的某种表现层；**
- **客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。**