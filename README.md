# 如何进行插件试开发

我们很多程序员在接到一项需求的时候，由于时间比较赶或者项目比较急或者程序历史遗留，总会写出让自己都觉得不满意的代码。

同样的功能，不一样的人来写，时间不一样，代码的质量也会不一样。尽管短时间内功能是Ok的，但是以后加新功能，或者需求改变了， 代码质量的好坏直接决定了新功能出来的时间和质量了。

最近自己写了一个node的模块，[通过手机号查询运营商以及号码归属地](https://github.com/navyxie/phone-service),整个程序写过3遍，虽然最后已经写成了插件式的形式，但是由于能力有限，总觉得好多地方存在不足，却又说不上来。下来就来跟大家分享一下这个模块的结构以及自己的想法。

## 存在的问题

暂时想到下面这些

1. 通过手机号查询运营商以及号码归属地这样的需求应该有很多个服务商或者api渠道可以调用，那么如何来管理这些API呢？
2. 每个API的请求方式可能都不一样，返回的结果也不一样，编码不一样，最终模块返回应当如何保证数据的一致性？
3. 当某一个API出现问题（比如请求被封掉或者不稳定）时，如何快速地踢出这个API，以最少改动，甚至不改动代码的情况下实现呢？
4. 当开发者使用这个模块时，觉得模块自身带的API不满足或者说自己有稳定的API可用，如何方便地扩展进来使用呢？
5. 这么多个API,在使用的时候应当分配哪些呢？
6. 如何做到快速地返回结果？
7. ...


## 模块结构

[附源码地址,Git>](https://github.com/navyxie/phone-service)

![模块结构](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoNtv06GOz5531vgZBqkjlOZyLUJ2gW4kictXMuyewZG2NAUC5j5OpGllg/0?wx_fmt=png)

## 源码解析

### [入口文件/lib/index.js](https://github.com/navyxie/phone-service/blob/master/lib/index.js)

> ![入口文件解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoNiakNqb1V8iaibyQhbwgxNibVMI3uTCsxlC4Etlkyf7f8wuYqjxx90Q6Ydg/0?wx_fmt=png) 
> ![入口文件解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoNtbBxyBsY8tUPdoic489c1mSfMAmT7nPzxoxKqBpvnG36W8kfQiawuASg/0?wx_fmt=png)
> ![入口文件解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoNWHxZhcmdS4hXJ82kib8koWDDibqxQGpseN9RrKQnlztUaDYVO7ac6ic9g/0?wx_fmt=png)

由上图我们可以看到，模块可由使用者配置参数。包括可以配置一次接口调用所需的API的并发数，保证接口以最快速度返回数据；使用者可以指定接口使用特定的插件。

### [插件管理/lib/plugin.js](https://github.com/navyxie/phone-service/blob/master/lib/plugin.js)

> ![插件管理解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoNUJNlY91BT1ibnlqvTn62niagWPmGm9aFQ7yib7FO7UK6StfyCQrUpVU3w/0?wx_fmt=png)

由上图我们可以看到，模块管理提供了基本的方法，添加模块，删除模块，执行模块等。

### [插件开发/lib/plugin.js](https://github.com/navyxie/phone-service/blob/master/plugin/taobao.js)

> ![插件开发解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoN1dP4SwqticDfDn2iasxaBu68ribs6f5jVwG6jEX0A0F2sKpBosia2WzCicw/0?wx_fmt=png)
> ![插件开发解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoN9heCia8kGUrcl2iaqCsUg1xLd3M0EjibskWS7qMkUGJfElm27icTbe0vOw/0?wx_fmt=png)

目前模块自带7个API，均以插件形式编写接入到模块中。由上图我们可以看到，插件开发极其简单，只需要定义插件名字（调用的时候需要用到），数据解析函数parse（因为不同api返回的结果不一样，所以需要统一处理）

## 测试

> ![测试解析](https://mmbiz.qlogo.cn/mmbiz/E7ia3F4UicMx8lXwibq1yX1Q2khFQs7WZoNjOXoE2Xiaa67Cf63x6JNsE3sKzGGlFzBPictHJHiapT1pL1SJNiaRcypNQ/0?wx_fmt=png)

当我们开发好一个插件时，如何保证一个插件是work的呢？那就需要通过测试了，写测试用例也比较简单（mocha,should）。


## 插件化开发的好处

+ 代码目录结构清晰
+ 代码可读性高
+ 代码可维护性强
+ 代码低耦合
+ 代码质量高
+ 多人独立开发，每个人负责单独的插件，互不影响
+ 某个插件出问题时，可快速删除，对用户以及开发者来说是透明的
+ 添加新的功能（这里主要指新的API渠道）时能够方便快速地接入
+ 培养程序员插件化开发的思维
+ ...

## 插件化开发适应场景

+ 某一个功能同时拥有多个渠道API或者第三方平台
+ 某一个功能具有多种实现
+ 应用越强越大越应当使用插件化开发
+ 当某些功能要提前开发好，隐藏在应用中时，可以通过插件化方式开启关闭（尤其是在开发Native app时，用户不需要升级app就能使用预先隐藏好的功能）
+ ...


#### 目前插件管理文件写得不是很好，还不够通用，需要继续完善，完整的源码请看[Git地址->](https://github.com/navyxie/phone-service)
