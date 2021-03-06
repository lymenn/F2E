
## 目录

* [输入url到展示经过了哪些步骤](#输入url到展示经过了哪些步骤)
* [三次握手](#三次握手)
* [四次挥手](#四次挥手)
* [https与http](#https与http)
* [http请求中的get和post请求方式的区别](#http请求中的get和post请求方式的区别)
* [常见的http状态](#常见的http状态)
* [TCP和UDP的区别](#TCP和UDP的区别)
* [跨域](#跨域)
* [缓存](#缓存)
* [http1.0、http1.1、http2.0](#http1.0、http1.1、http2.0)
* [websocket](#websocket)
* [Preload、prefetch](#Preload、prefetch)
* [cookie和session](#cookie和session)
* [JWT](#JWT)
* [CDN](#CDN)
* [nds](#dns在哪里)

## 内容


> 1.输入url到展示经过了哪些步骤

<a name="输入url到展示经过了哪些步骤"></a>

[http请求经过哪些步骤](https://www.cnblogs.com/slivens/p/12902145.html)

* 域名解析 
* 发起TCP的3次握手
* 建立TCP连接后发起http请求
* 服务器响应http请求，浏览器得到html代码
* 浏览器解析html代码，并请求html代码中的资源（如js、css、图片等）
* 浏览器对页面进行渲染呈现给用户

* 浏览器会查看当前的url有没有dns缓存，没有回到系统查看是否有dns缓存，host文件查看对应域名
* 如果有缓存直接读取缓存，没有缓存进行dns查询，会向本地配置的dns服务器发起请求
* 查询到对应ip后，可能是cdn资源，这时目标服务器是一台cdn负载均衡服务器，cdn根据用户信息返回离其最近的节点
* 向最近节点服务器简历tcp链接，三次握手后，目标服务器根据参数，url链接返回不同的资源，
* 目标服务器可能是台代理服务器，nginx代理服务器
* 在nginx服务器进行中间层转发，最终到达目标服务器，目标服务器、参数不同进行不同的处理，最终返回对应的资源
* 如果返的是html，浏览器会对html进行解析，解析成对应的渲染树
* 开始渲染前会将head中的资源加载进行，如果是css会生成对应的样式渲染树，遇到script脚本会根据脚本加载特性的不同进行不同的处理
* 如果没有带defer、async属性会加载完脚本后在进行渲染步骤，如果是async属性会边加载边进行渲染，


> 2.三次握手

<a name="三次握手"></a>


三次握手的目的：保证客户端和服务端都具有，发送、接收数据的能力

三次握手指的是传输层的Tcp协议为保证数据传输有效性的的策略：

开始阶段：客户端状态为closed、服务端为listen（监听状态）

* 第一次握手：客户端向服务端发送SYN=1（建立联机），报文中指定一个序列号（seq num），表明想要建立连接的想法。此时客户端处于SYN_send状态
* 第二次握手：服务端在接受到客户端的SYN，将客户端的序列号+1作为ASK作为回应，并在报文中返回服务端的序列号，返回ASK、SYN（同步标识），作为响应。表明服务端已经准备好可以建立连接了，此时服务端处于SYN_RCVD状态，半连接队列，若在指定时间内没有收到客户端响应，会重新回应，到底指定最大时间会进行重连
* 第三次握手：客户端在收到服务端的ACK后，将服务端的ISN+1作为ASK回应，表明可以通信了，客户端此时处于established

服务端在收到客户端的ASK后，状态变为established，后续双方开始通信



> 3.四次挥手

<a name="四次挥手"></a>

双方都处于established状态

* 第一次挥手客户端发送FIN报文，报文中会指定一个序列号。此时客户端处于FIN_WAIT1-1状态
* 第二次握手服务端在接收到FIN之后，将客户端的序列号+1作为ACK报文，发送给客户端表明已经接收到客户端断开连接的请求了，此时服务端处于CLOASE_WAIT
* 第三次挥手服务端也准备好断开连接了，会发送FIN，并指定一个序列号在报文发送给客户端，此时服务端处于LAST_WAIT状态
* 第四次挥手，客户端在接收到服务端的FIN后，一样会发一个ACK报文作为回应，并且把服务端的序列号值+1作为ACK报文的序列值，此时客户端处于TIME_WAIT状态，需要过一整子确保服务端收到自己的ACK后，才会进入CLOSED状态。这里为什么是TIME_WAIT,因为服务端没有收到客户端的ACK，会过一段时间重发FIN报文给客户端，客户端收到ASK就知道之前的ASK报文丢失了，就会重新发送ACK给服务端了

服务端收到ACK后就关闭了

> 4.https与http

<a name="https与http"></a>

[https](https://juejin.im/post/59e4c02151882578d02f4aca#heading-5)
[https详细交互过程](https://www.jianshu.com/p/42e1c073c142)
![](https://user-gold-cdn.xitu.io/2018/5/21/1638197d96d391ca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


为什么会出现https，因为http都是以明文的方式进行传输的，容易被读取修改

https=http+ssl(secure scokets layer)，安全协议、是可以进行加密、身份认证的协议

SSL协议是位于http之下，TCP之上的协议

* HTTPS采用非对称加密，如何减少加解密时间
    * 对称加密：加密和解密采用同一个秘钥，加解密速度快
    * 非对称加密：分为公钥和秘钥，公钥加密私钥解密，私钥加密，公钥解密，加解密速度慢
* 解决加解密慢的问题，非对称加密和对称加密混合使用：
    * 服务端对将要是用什么加密算法和秘钥通过私钥加密，传输给客户端
    * 客户端使用公钥对加密的内容进行解密，得到对称加密算法和私钥
    * 后续通过对称加密算法进行加解密

* 如何保证公钥不被串改，服务端在发送公钥的时候可能被第三方拦截，返回给客户端自己的公钥

* SSL/TSL协议交互过程：
    * 客户端向服务端索要并验证公钥
    * 双方协商生成对话秘钥
    * 双方采用对话秘钥通信
* 详细的交互过程
    * 客户端生成一串随机数，稍后用于生成对话秘钥。向服务端发送支持的加密算法如：RSA、TSL协议版本
    * 服务端确定TSL协议的版本，如果客户端不支持协议关闭加密通信。生成一串随机数，稍后用于生成对话秘钥，确定使用的TSL版本，向客户端发送服务端证书
    * 客户端收到服务端的回应后，会验证服务端证书，如果服务端证书不是可信机构、证书过期或者证书上的域名与实际域名不一致、就会向访问者显示一个警告，确认是否继续通信。如果证书没问题，客户端会从证书中取出服务端的公钥，然后向服务端发送三条信息：一个随机数使用公钥加密，编码发送时改变：随后都用双方协商的加密方法和秘钥发送。如果服务端需要客户端提供证书，这时也会发送证书
    * 服务端最后的响应：编码改变通知，表示随后的信息都将用双方协定的加密方式进行通信握手阶段结束通知
    * 接下来，客户端和服务端进行加密通信，使用的就是完完全全的HTTP协议，只不过用“会话秘钥”加密内容
* 简介http交互过程：
    * 客户端向服务端发送https请求
    * 服务端返回CA证书，CA证书内包括公钥。客户端验证CA证书是否有效，域名、有效时间等信息是否合规
    * 客户端生成对称加密秘钥，通过公钥加密传输给服务端，服务端返回信息表示后续通过对称加密信息进行通信
    * 进行加密通信


> 5.http请求中的get和post请求方式的区别

<a name="http请求中的get和post请求方式的区别"></a>


* get支持传输的数据类型较少只能通过url上拼接参数传递，无法传输字节流，post请求，支持传输的数据类型多，包括json、form表单，二进制文件
* get请求在进行跨域请求时浏览器不会触发options请求，post在跨域请求时，会触发options请求
* get传输的数据类型取决于浏览器地址的长度，post传输的数据量比get更大
* get请求会主动被浏览器cache，像资源文件这一类都是通过get请求获取的。post请求不会被cache
* get请求在浏览器回退时对浏览器来说是无害的不会重新请求，post会重新触发请求
* get相对来说比post不安全，因为敏感信息暴露在url中

* get和post没有本质上的区别，两者都是http协议中的两种请求方式
* get也可以在body中添加数据，不过不建议这么做
* get请求只发送一个tcp数据包、而post请求会发送两个数据包，get请求直接发送head 和data、而post会先发送head，服务端返回100状态（继续）时，才会发送剩余的data。当然在不同环境下实现的也不同
* 在浏览器上的区别：
    * 在浏览器回退或刷新的时候get请求是无害的，
    * 由于在浏览器上对请求的url长度是限制的，并且get请求在web端时没法在body添加数据，但是post请求可以在body中添加请求数据，所以post请求传输的数据比get请求更多，而且数据类型更广
    * 浏览器中get只能通过url上增加参数



> 6.常见的http状态

<a name="常见的http状态"></a>

* 100（继续）
* 200（服务端成功响应）
* 301（永久重定向）包括location、302（临时重定向）、304（没有修改，从缓存读取）
* 400（参数错误、语义错误）、401（无权访问）、403（请求被拒绝了）、404找不到资源了、405（请求方式被拒绝了）
* 500（服务端出现未预料的错误）、503（服务端无法处理当前请求）、504（服务端超时了）


> 7.TCP和UDP的区别

<a name="TCP和UDP的区别"></a>

* TCP协议是面向连接的协议，为保证数据可靠性，它有三次握手、四次挥手的策略。只能一对一通信
* UDP协议不是面向连接的，接收端和发送端不建立连接，发送方需要发送数据时，会直接向接收方发送数据，传输速度和时效很高，但不能保证数据是否正确传输，支持一对一或一对多通信

> 8.跨域

<a name="跨域"></a>

* 什么是跨域，什么时候回出现跨域需求：
    * 跨域是出于浏览器的同源策略限制，它限制了一个源的文档或脚本如何与另一个文档资源交互
    * 什么时候回出现跨域：协议、端口、主机不同时会出现跨域
    * 不同职责的服务放在不同的工程中，不同的工程会对应不同的域名，因此就会出现跨域问题
* 如何解决跨域：
    * jsonp的形式，因为浏览器允许跨域资源嵌入，像script、img标签。通过动态插入script标签来获取数据的方式
    * CORS是一种新的官方方案，使服务器支持跨域请求
        * 在服务器返回头中携带Access-Control-Allow-Origin: *
        * Access-Control-Allow-Methods: get、post、options
        * 表明支持跨域的域名，如果当前页面域名包含在支持的域名中，浏览器就不会拦截
        * CORS请求分为简单请求和非简单请求：
            * 简单请求：GET、HEAD、POST（content-type:text/plain、multipart/form-data、application/x-www-form-urlencoded），且没有人为设置首部字段集合之外的其他首部字段
            * 非简单请求，人为设置首部字段集合之外的其他首部字段
                * content-type不属于（content-type:text/plain、multipart/form-data、application/x-www-form-urlencoded）
            * Access-Control-Allow-Credentials 为true，浏览器才会把cookie响应结果传递给客户端程序
    * POSTmessage，跨窗口进行通信

> 9.缓存

<a name="缓存"></a>

* 浏览器缓存
    * 缓存get请求200后的资源例如：HTML文档、图片、js，将这些资源缓存在本地
    * 用户点击返回按钮、等操作直接从磁盘读取，减少时间，增加用户体验
    * 浏览器对于缓存的处理是根据，第一次请求资源时返回的响应头来确定的
    * from disk
        * from dist or from memory
        * 内存使用率很高会放在磁盘中，内存使用率很高放在memory中
    * 响应头缓存字段
        * Cache-Control（HTTP/1.1定义的关于缓存的字段，优先级大于Expires):
            * no-cache、no-store：强制浏览器验证、不缓存
            * max-age=2592000： 缓存的时长，秒为单位。它规定了过期的相对时间
            * min-fresh：期望在指定时间内任有效
            * only-if-cache：从缓存中读取
            * no-transform: 代理不可更改类型
        * Expires: 它规定了缓存过期的一个绝对时间（http1.0中定义的缓存字段），由于生成的时间是服务端的时间，判断的确实客户端的时间，客户端可以更改时间，所以后面引入了max-age
        * last-Modified: 表示文件最后一次修改的时间
        * ETag: 1.1新增，解决last-modified只能对秒级别的改动检测，文件内容的md5值，hash值。通过判断内容hash，来决定是否需要返回304
    * 强制缓存：
        * 当浏览器访问资源响应头返回了expires或Cache-control:max-age=t(s)
        * 当有Cache-control:max-age=t(s)时，以date加max-age与当前时间进行判断查看是否过期，没过期直接读取缓存资源
        * 没有Cache-control有exprise，通过exprise与当前时间进行判断
    * 协商缓存：
        * 利用last-Modified、ETag浏览器会进入协商缓存阶段
        * 会在本次请求头里携带：If-Moified-Since（为last-Modified）、If-None-Match（为ETag）这两个字段，服务器通过这两个字段来判断文件是否有修改
        * 如果有修改会返回200和新的内容，没修改会返回304从缓存中直接读取
    * Pragma(可以指定缓存策略): 
        * `<meta http-equiv="Pragma" content="no-cache">`
    
* appacache
    * 申明mainappcache文件，在里面可以列出哪些文件缓存，离线状态或出错指定404
    * 标准已废弃
* 代理服务器缓存

> 10.http1.0、http1.1、http2.0

<a name="http1.0、http1.1、http2.0"></a>

* http1.0和http1.1
    * 缓存：由原来的expirse、last-modify增加etag、if-none-match提供更多缓存头来控制缓存
    * 带宽优化：1.0时会传输不需要的对象内容，且不支持断点传输。1.1支持断点传输，且只传输需要的部分
    * 错误通知：1.1新增了多个错误状态码
    * host头处理：1.1header增加了host，认为一个域名可能存在多个ip，可以指定机器
    * 长连接：1.1支持长连接，在TCP连接上可以传递多个http请求和响应减少了建立和关闭连接的消耗和延迟，在1.1中默认开启connection:keep-alive
* http1.x和http2.0
    * 格式：http2.0采用二进制格式
    * 效率：http2.0采用多路复用，一个连接可以多应多个http请求
    * header压缩：采用header压缩来减少header的大小
    * 服务端推送：http2.0服务端可以采取主动推送的方式,但是只能推送静态资源
    * http1.1长连接和http2.0多路复用有什么关系
        * 长连接是使用队列的形式，建立一个tcp连接后，http按照队列阻塞的形式向服务端通信，某个任务比较耗时阻塞其他任务
        * 多路复用可以在一个tcp连接上并行请求，相互之间不会受到影响
    * http1.1为什么不能实现多路复用
        * [http2.0](https://juejin.im/post/5d70848df265da03d871de9c#heading-2)
        * [http2.0问题](https://juejin.im/post/5c0ce870f265da61171c8c66)
        * http1.1：
            * 基于文本分割解析协议，以换行符key:value分割每一行
            * 一次只能处理一个请求，分隔符分割消息数据，在完成前不能停止解析，然后进行下一个请求
            * 解析数据无法预知需要多大内存
        * http2.0
            * http2.0是通过二进制分帧的形式，帧是数据单元实现了数据的封装
            * 你可以通过解析前面了解需要多少字节处理
            * 请求和响应可以交错，
            * 每一个TCP链接中承载了多个双向的流，每个流都有独一无二的标识
            * 而流就是由二进制帧组成的，接收端组装成完整的数据
    * http2.0什么时候性能比http1.1性能更差
        * http2.0发生丢包，因为多个接口都拆分了多个部分，
        * 多个部分可能都发生丢包，导致多个接口受到影响

> 11.websocket

<a name="websocket"></a>

* 来源：
    * socket不是协议，它是基于TCP/IP传输协议上的一层封装，可以理解为对TCP/IP的简单抽象
    * Websocket是包装了应用层面上的socket，他实现了服务端和浏览器的双工通信，实现了server push
    * 为解决http只能由客户端发起通信的问题，可以主动推送消息到客户端
    * http单项请求的特点，服务端如果出现数据变化，客户端只能通过轮询的方式明白服务端发生了变化
* 握手：
    * webSocket在建立http连接后会进行两次握手
    * 客户端发送在报文中发送upgrade: websocket，
    * 服务端返回101,switch protocol，后续可以双工通信
* 事件：
    * onopen（开启）、message（接收消息）、err（错误）、close（关闭）
* 心跳机制：
    * 有可能某一方因为一场情况断开连接，在报错或者断开连接时请求连接
    * 在建立连接后，每过一段时间向服务端发送'ping'服务端会返回'pong'确定双方还在通信。
    * 若一段时间内服务端没返回pong，客户端断开连接，重连


> 12.Preload、prefetch

<a name="Preload、prefetch"></a>

* link-type
    * rel=`stylesheet`

* Preload
    * [详解preload](https://www.jianshu.com/p/24ffa6d45087)
    * 提前加载资源，等到需要执行才执行
    * 不会影响onload事件，
    * 通过link标签，增加ref="preload"，通过http头增加rel="preload"
    * 提前加载路由文件，切换时才执行
    * `<link rel="preload" href="./test.css" as="style" onload="this.rel='stylesheet'">`
    * 通过更改ref属性使其生效
    * as：
        * "script"、"style"、"image"、"media"、"document"
* prefetch
    * 资源优先级比较低

> 13.cookie和session

<a name="cookie和session"></a>

* session存在于服务端，在用户第一次请求时生成session
* 将sessionId放在cookie中返回给浏览器，后续http请求会携带cookie，cookie中存在sessionid
* 服务端根据对应的sessionId找到对应的session，进行会话记录

[cookie和session的区别](https://www.zhihu.com/question/19786827)

* session与cookie的区别：
    * session存储在服务端，cookie存储在客户端
    * 存储类型不同：session可以存储任何类型，cookie只能存储字符串
    * 有效期不同：session一般失效时间较短，cookie可以长时间存储
    * 存储大小不同：session可以存储量较大，cookie限制在4kb内

> 14.JWT

<a name="JWT"></a>


* 全称：json web token，通过生成token
* 通过token代替session，服务端不用存储会话
* 可以有效的方式CSRF跨站追踪攻击


> 15.CDN

<a name="CDN"></a>

[深入了解cdn](https://juejin.im/post/59ba146c6fb9a00a4636d8b6)

* cdn（content delivery network）
    * 将网站内容发送到最接近用户的网络“边缘”
    * 用户可就近取得用户内容，提高网站的响应速度
    * 比喻CDN=镜像+cache
* cdn工作流程
    * 访问某个静态资源，根据资源的域名会被指向cdn全局的cdn负载均衡服务器
    * 由负载均衡服务器来最终分配是哪个地方的访问用户，返回给用户最近的cdn节点
    * 用户最后直接去这个cdn节点访问静态资源，如果这个节点资源部存在，就会回到源站找资源

> 16.dns在哪里

<a name="dns在哪里"></a>

* DNS域名解析采用的是递归查询的方式，过程是
* 先去找DNS缓存->缓存找不到就去找本地域名服务器->根域名服务器->根域名->com域名服务器，一层层查找
* 这样递归查找之后，找到了，给我们的web浏览器


先从缓存（浏览器缓存，操作系统缓存、路由器缓存、以及isp服务商缓存），找不到的话就去根域名服务器->顶级域名服务器->二级域名依次的去查询么