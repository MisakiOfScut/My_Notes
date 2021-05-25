# HTTP

HTTP 是一个在计算机世界里专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范。不仅仅适用于[服务器<-->客户端]也是适用于[服务器<-->服务器]

### HTTP 状态码

<div align=center> <img width=500, height=300, src="https://github.com/MisakiOfScut/LearningNote/raw/master/1.%E9%9D%A2%E8%AF%95%E6%80%BB%E7%BB%93/image/Http%E7%8A%B6%E6%80%81%E7%A0%81.webp"></div>

+ 1xx   
1xx 类状态码属于提示信息，是协议处理中的一种中间状态，实际用到的比较少。  
+ 2xx  
<font color=red>2xx 类状态码表示服务器成功处理了客户端的请求 </font>，也是我们最愿意看到的状态。
  
  + **「200 OK」** 是最常见的成功状态码，表示一切正常。如果是非 HEAD 请求，服务器返回的响应头都会有 body 数据。
  + 「**204 No Content**」也是常见的成功状态码，与 200 OK 基本相同，但响应头没有 body 数据。
  + 「206 Partial Content」是应用于 HTTP 分块下载或断电续传，表示响应返回的 body 数据并不是资源的全部，而是其中的一部分，也是服务器处理成功的状态。
+ 3xx  
3xx 类状态码表示客户端请求的资源发送了变动，需要客户端用新的 URL 重新发送请求获取资源，也就是重定向。
  
  + 「301 Moved Permanently」表示永久重定向，说明请求的资源已经不存在了，需改用新的 URL 再次访问。
  + 「302 Moved Permanently」表示临时重定向，说明请求的资源还在，但暂时需要用另一个 URL 来访问。
    
    > 301 和 302 都会在响应头里使用字段 Location，指明后续要跳转的 URL，浏览器会自动重定向新的 URL。
  + 「304 Not Modified」不具有跳转的含义，表示资源未修改，重定向已存在的缓冲文件，也称缓存重定向，用于缓存控制。
+ **4xx**  
  <font color=red> 4xx 类状态码表示客户端发送的报文有误</font>，服务器无法处理，也就是错误码的含义。
  
  + **「400 Bad Request」** 表示客户端请求的报文有错误，但只是个笼统的错误。
  + **「403 Forbidden」** 表示服务器禁止访问资源，并不是客户端的请求出错。
  + **「404 Not Found」** 表示请求的资源在服务器上不存在或未找到，所以无法提供给客户端。
+ 5xx  
    <font color=red> 5xx 类状态码表示客户端请求报文正确，但是服务器处理时内部发生了错误</font>，属于服务器端的错误码。
    
    + **「500 Internal Server Error」** 与 400 类似，是个笼统通用的错误码，服务器发生了什么错误，我们并不知道。
    + 「501 Not Implemented」表示客户端请求的功能还不支持，类似“即将开业，敬请期待”的意思。
    + 「502 Bad Gateway」通常是服务器作为网关或代理时返回的错误码，表示服务器自身工作正常，访问后端服务器发生了错误。
    + 「503 Service Unavailable」表示服务器当前很忙，暂时无法响应服务器，类似“网络服务正忙，请稍后重试”的意思

### HTTP 字段
+ ***Host***: 客户端发送请求时，用来指定服务器的域名。
+ ***Content-Length***：服务器在返回数据时，会有 Content-Length 字段，表明本次回应的数据长度。
+ ***Connection***：Connection 字段最常用于客户端要求服务器使用 TCP 持久连接，以便其他请求复用。
  
  > HTTP/1.1 版本的默认连接都是持久连接，但为了兼容老版本的 HTTP，需要指定 Connection 首部字段的值为 Keep-Alive。  
  >
  > *Connection: keep-alive*
+ ***Content-Type***：用于服务器回应时，告诉客户端本次数据是什么格式。
  
  >Content-Type: text/html; charset=utf-8  ：表明，发送的是网页，而且编码是UTF-8。  
  > 
  >客户端请求的时候，可以使用 **Accept** 字段声明自己可以接受哪些数据格式。比如：  
  >  Accept: */* ：客户端声明自己可以接受任何格式的数据。
+ ***Content-Encoding***：说明数据的压缩方法。表示服务器返回的数据使用了什么压缩格式
  
  > 客户端在请求时，用 **Accept-Encoding** 字段说明自己可以接受哪些压缩方法。

### 请求方法：GET、POST
**区别**:  

+ Get 方法的含义是 <font color=red>请求从服务器获取资源</font>，这个资源可以是静态的文本、页面、图片视频等。
+ POST 方法则是相反操作，<font color=red>它向 URI 指定的资源提交数据</font>，数据就放在报文的 body 里。

GET请求是只读属性，请求内容直接在请求行中不安全。POST是可以修改服务器的数据，内容在body里比较安全。


### HTTP 特点
#### 优点   

HTTP 最凸出的优点是「简单、灵活和易于扩展、应用广泛和跨平台」。

+ 简单：HTTP 基本的报文格式就是 header + body ，头部信息也是 key-value 简单文本的形式，易于理解。

+ 灵活和易于扩展：HTTP 协议里的各类请求方法、URI/URL、状态码、头字段等每个组成要求都没有被固定死，都允许开发人员自定义和扩充。同时 HTTP 由于是工作在应用层（ OSI 第七层），**则它下层可以随意变化**。
  
    > 比如，HTTPS 也就是在 HTTP 与 TCP 层之间增加了 SSL/TLS 安全传输层，HTTP/3 甚至把 TCP 层换成了
基于 UDP 的 QUIC。


#### 缺点 

无状态、明文传输、不安全

+ 无状态：无状态的坏处使得务器没有记忆能力，它在完成有关联性的操作时会非常麻烦

  > 对于无状态的常用解决方案是Cookie
  >
  > Cookie 通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。相当于，在客户端第一次请求后，服务器会下发一个装有客户信息的「小贴纸」，后续客户端请求服务器的时候，带上「小贴纸」，服务器就能认得了了
  >
  > 
  >
  > Cookie 与 Session 区别
  >
  > cookie是存储在本地浏览器，而session存储在服务器。存储在服务器的数据会更加的安全，不容易被窃取。但存储在服务器也有一定的弊端，就是会占用服务器的资源
  >
  > 
  >
  > cookie 与 session 的配合使用：
  >
  > + 存储在服务端：通过 cookie 存储一个 session_id，然后具体的数据则是保存在 session中。如果用户已经登录，则服务器会在 cookie 中保存一个 session_id，下次再次请求的时候，会把该 session_id 携带上来，服务器根据 session_id 在 session 库中获取用户的session 数据。就能知道该用户到底是谁，以及之前保存的一些状态信息。这种专业术语叫做  **server-side session**。
  > + 将 session 数据加密，然后存储在 cookie 中。这种专业术语叫做 **client-side session**。
+ 明文传输

  明确传输利于阅读和DEBUG，但是信息不安全，在信息传输过程中信息内容毫无隐私可言。

+ 不安全   
  + 通信使用明文（不加密），内容可能会被窃听
  + 不验证通信方的身份，因此有可能遭遇伪装
  + 无法证明报文的完整性，所以有可能已遭篡改

  HTTP 的不安全性问题可以使用 HTTPs 引入 SSL/TLS 层得到解决

### HTTPs
https中的 **s** 指的是 **security**。https协议是在http基础上加上ssl的应用层协议，增加了安全性和可靠性。

#### HTTPs 的建立过程
SSL/TLS 协议基本流程：
+ 客户端向服务器索要并验证服务器的公钥
+ 双方协商生产「会话秘钥」
+ 双方采用「会话秘钥」进行加密通信

前两步也就是 SSL/TLS 的建立过程，也就是握手阶段。SSL/TLS 的「握手阶段」涉及四次通信，可见下图：

<div align=center> <img src=https://github.com/MisakiOfScut/LearningNote/raw/master/1.%E9%9D%A2%E8%AF%95%E6%80%BB%E7%BB%93/image/HTTPS%E5%9B%9B%E6%AC%A1%E6%8F%A1%E6%89%8B%E8%BF%87%E7%A8%8B.jpg></div> 

#### HTTP与HTTPS的区别是什么
+ HTTP端口号80端口， HTTPS端口号是443端口
+ HTTP是明文传输，HTTPS是密文传输。
+ HTTP 连接建立相对简单， TCP 三次握手之后便可进行 HTTP 的报文传输。而 HTTPS 在 TCP三次握手之后，还需进行 SSL/TLS 的握手过程，才可进入加密报文传输
+ HTTPS 协议需要向 CA（证书权威机构）申请数字证书，来保证服务器的身份是可信的。
#### HTTPS的实现原理
+ 混合加密的方式实现信息的机密性，解决了窃听的风险。
  
  HTTPS 采用的是对称加密和非对称加密结合的「混合加密」方式：
  + 在通信建立前采用非对称加密的方式交换「会话秘钥」，后续就不再使用非对称加密。
  + 在通信过程中全部使用对称加密的「会话秘钥」的方式加密明文数据。

  采用「混合加密」的方式的原因：
  + 对称加密只使用一个密钥，运算速度快，密钥必须保密，无法做到安全的密钥交换。
  + 非对称加密使用两个密钥：公钥和私钥，公钥可以任意分发而私钥保密，解决了密钥交换问题但速度慢

+ 摘要算法的方式来实现完整性，它能够为数据生成独一无二的「指纹」，指纹用于校验数据的完整性，解决了篡改的风险

  客户端在发送明文之前会通过摘要算法算出明文的「指纹」，发送的时候把「指纹 + 明文」一同 加密成密文后，发送给服务器，服务器解密后，用相同的摘要算法算出发送过来的明文，通过比较客户端携带的「指纹」和当前算出的「指纹」做比较，若「指纹」相同，说明数据是完整的
  
+ 将服务器公钥放入到数字证书中，解决了冒充的风险

### HTTP迭代

#### HTTP1.0 到 HTTP 1.1

+ 长连接：

  早期 HTTP/1.0 性能上的一个很大的问题，那就是每发起一个请求，都要新建一次 TCP 连接（三次握手），而且是串行请求，做了无谓的 TCP 连接建立和断开，增加了通信开销。为了解决上述 TCP 连接问题，HTTP/1.1 提出了长连接的通信方式，也叫持久连接。这种方式的好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销，减轻了服务器端的负载。

+ 管道网络传输

  这个类似于TCP的窗口，后一个请求不需要等待前一个请求回来才能发送出去，而是可以一起发送出去。

  > 但是响应顺序还是先来先服务，如果前面的请求阻塞了，那么还是会使得后面的请求得不到处理，这叫“队头阻塞”

#### HTTP1.1 到 HTTP2.0 

**HTTP/2 协议是基于 HTTPS 的**，所以 HTTP/2 的安全性也是有保障的。那 HTTP/2 相比 HTTP/1.1 性能上的改进：

+ 头部压缩  
  HTTP/2 会压缩请求头（*Header*）。如果你同时发出多个请求，他们的请求头是一样的或是相似的，那么，协议会帮你消除重复的部分。

  这就是所谓的 **HPACK** 算法：在客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就**提高速度**了。
+ 二进制格式   
   HTTP/2 不再像 HTTP/1.1 里的纯文本形式的报文，而是全面采用了二进制格式，头信息和数据体都是二进制，并且统称为帧（frame）：头信息帧和数据帧。

   <div align=center> <img src=https://github.com/MisakiOfScut/LearningNote/raw/master/1.%E9%9D%A2%E8%AF%95%E6%80%BB%E7%BB%93/image/HTTP_2_%E6%A0%BC%E5%BC%8F.jpg></div> 


   对于计算机而言，无需再将明文的报文转成二进制，而是直接解析二进制报文，这**增加了数据传输的效率**。

+ 数据流     
  HTTP/2 的数据包不是按顺序发送的，同一个连接里面连续的数据包，可能属于不同的回应。因此，必须要对数据包做标记，指出它属于哪个回应。

  每个请求或回应的所有数据包，称为一个数据流（`Stream`）。每个数据流都标记着一个独一无二的编号，其中规定客户端发出的数据流编号为奇数， 服务器发出的数据流编号为偶数、客户端还可以指定数据流的优先级。优先级高的请求，服务器就先响应该请求
+ **多路复用**  
    HTTP/2 是可以在一个连接中并发多个请求或回应，而**不用按照顺序一一对应**。移除了 HTTP/1.1 中的串行请求，不需要排队等待，也就不会再出现「队头阻塞」问题，降低了延迟，大幅度**提高了连接的利用率**。
+ **服务器推送**    
    HTTP/2 还在一定程度上改善了传统的「请求 - 应答」工作模式，服务不再是被动地响应，也可以主动向客户端发送消息。