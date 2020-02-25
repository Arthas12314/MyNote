# HTTP & Servlet
## 简介
* 协议:双方在交互、通讯的时候， 遵守的一种规范、规则。
* http协议 针对网络上的客户端 与 服务器端在执行http请求的时候，遵守的一种规范。 其实就是规定了客户端在访问服务器端的时候，要带上哪些东西， 服务器端返回数据的时候，也要带上什么东西。 
## HTTP请求数据解释
- 请求行
	* POST /examples/servlets/servlet/RequestParamExample HTTP/1.1 
	POST ： 请求方式 ，以post去提交数据
	/examples/servlets/servlet/RequestParamExample 请求的地址路径，就是要访问哪个地方
	HTTP/1.1 协议版本
- 请求头
	Accept: application/x-ms-application, image/jpeg, application/xaml+xml, image/gif, image/pjpeg, application/x-ms-xbap, */*
	Referer: http://localhost:8080/examples/servlets/servlet/RequestParamExample
	Accept-Language: zh-CN
	User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)
	Content-Type: application/x-www-form-urlencoded
	Accept-Encoding: gzip, deflate
	Host: localhost:8080
	Content-Length: 31
	Connection: Keep-Alive
	Cache-Control: no-cache

	Accept: 客户端向服务器端表示能支持什么类型的数据。 
	Referer ： 真正请求的地址路径，全路径
	Accept-Language: 支持语言格式
	User-Agent: 用户代理 向服务器表明，当前来访的客户端信息。 
	Content-Type： 提交的数据类型。经过urlencoding编码的form表单的数据
	Accept-Encoding： gzip, deflate ： 压缩算法 。 
	Host ： 主机地址
	Content-Length： 数据长度
	Connection : Keep-Alive 保持连接
	Cache-Control ： 对缓存的操作
- 请求体
	
	* 浏览器真正发送给服务器的数据
## HTTP响应数据解析
> 响应的数据里面包含三个部分内容 ： 响应行 、 响应头 、响应体
	HTTP/1.1 200 OK
	Server: Apache-Coyote/1.1
	Content-Type: text/html;charset=ISO-8859-1
	Content-Length: 673
	Date: Fri, 17 Feb 2017 02:53:02 GMT
	...这里还有很多数据...
- 响应行
	HTTP/1.1 200 OK
	协议版本
	状态码
		本次交互到底是什么样结果的一个code. 		
		200 : 成功，正常处理，得到数据。	
		403  : forbidden  拒绝
		404 ： Not Found
		500 ： 服务器异常
	OK
		对应前面的状态码
- 响应头
	Server:  服务器是哪一种类型。  Tomcat	
	Content-Type ： 服务器返回给客户端你的内容类型
	Content-Length ： 返回的数据长度
	Date ： 通讯的日期，响应的时间	
- 响应体
## HTTP方法

>客户端发送的 请求报文 第一行为请求行，包含了方法字段

- GET 获取资源 
当前网络请求中，绝大部分使用的是 GET 方法
- HEAD 获取报文首部 
和 GET 方法类似，但是不返回报文实体主体部分。主要用于确认 URL 的有效性以及资源更新的日期时间等
- POST 传输实体主体
POST 主要用来传输数据，而 GET 主要用来获取资源
- PUT 上传文件
由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法
```
	PUT /new.html HTTP/1.1
	Host: example.com
	Content-type: text/html
	Content-length: 16	
	<p>New File</p>
```
- PATCH 对资源进行部分修改
PUT 也可以用于修改资源，但是只能完全替代原始资源，PATCH 允许部分修改
- DELETE 删除文件
与 PUT 功能相反，并且同样不带验证机制
- OPTIONS 查询支持的方法
查询指定的 URL 能够支持的方法。会返回```Allow:GET,POST,HEAD,OPTIONS```这样的内容
- CONNECT 要求在与代理服务器通信时建立隧道
使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输
```CONNECT www.example.com:443 HTTP/1.1```
- TRACE 追踪路径
服务器会将通信路径返回给客户端。
发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器就会减 1，当数值为 0 时就停止传输。通常不会使用 TRACE，并且它容易受到XST攻击（Cross-Site Tracing，跨站追踪）
### GET与POST区别
- POST
	* 数据是以流的方式写过去，不会在地址栏上面显示。  现在一般提交数据到服务器使用的都是POST
	* 以流的方式写数据，所以数据没有大小限制
- GET
	* 会在地址栏后面拼接数据，所以有安全隐患。 一般从服务器获取数据，并且客户端也不用提交上面数据的时候，可以使用GET
	* 能够带的数据有限， 1kb大小
#### 具体比较
- 作用 
GET 用于获取资源，而 POST 用于传输实体主体
- 参数
	* GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看
	* 因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 中文 会转换为 %E4%B8%AD%E6%96%87，而空格会转换为 %20。POST 参数支持标准字符集
- 安全
	* 安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的
	* GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变
	* 安全的方法除了 GET 之外还有：HEAD、OPTIONS 不安全的方法除了 POST 之外还有 PUT、DELETE
- 幂等性
	* 幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）
	* 所有的安全方法也都是幂等的
	* 在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是
- 可缓存
如果要对响应进行缓存，需要满足以下条件
	* 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的
	* 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501
	* 响应报文的 Cache-Control 首部字段没有指定不进行缓存
- XMLHttpRequest
	* 在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会
	* 而 GET 方法 Header 和 Data 会一起发送
[浅谈get与post的区别](https://www.cnblogs.com/hyddd/archive/2009/03/31/1426026.html)
## HTTP状态码

>服务器返回的 响应报文 中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果

![HTTP状态码](..\笔记图片\http状态码.png)

- 1XX 表示信息
	* 100 Continue ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应
- 2XX 表示成功
	* 200 OK
	* 204 No Content ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用
	* 206 Partial Content ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容
- 3XX 表示重定向
	* 301 Moved Permanently ：永久性重定向
	* 302 Found ：临时性重定向
	* 303 See Other ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源
		+ 虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法
	* 304 Not Modified ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码
	* 307 Temporary Redirect ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法
- 4XX 表示客户端错误
	* 400 Bad Request ：请求报文中存在语法错误
	* 401 Unauthorized ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败
	* 403 Forbidden ：请求被拒绝
	* 404 Not Found
- 5XX 表示服务器错误
	* 500 Internal Server Error ：服务器正在执行请求时发生错误
	* 503 Service Unavailable ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求
## HTTP基本认证过程

* 发送请求：如GET/admin/http1.1 Host：fengluo.com
* 返回状态码401告知客户端验证：http 401 Authorization required
* 用户名和密码通过base64编码后发送 GET/admin HTTP/1.1 Host：fengluo.com Authorization：Basic xxxxxxxxxxxxxxx
* 成功返回状态码200，失败返回401继续认证

## 具体应用

- 长连接和短连接
	* 当浏览器访问一个包含多张图片的 HTML 页面时，除了请求访问的 HTML 页面资源，还会请求图片资源。如果每进行一次 HTTP 通信就要新建一个TCP 连接，那么开销会很大。长连接只需要建立一次 TCP 连接就能进行多次 HTTP 通信
		+ 从 HTTP/1.1 开始默认是长连接的，如果要断开连接，需要由客户端或者服务器端提出断开，使用 Connection : close;
		+ 在 HTTP/1.1 之前默认是短连接的，如果需要使用长连接，则使用 Connection : Keep-Alive;
	* 默认情况下，HTTP 请求是按顺序发出的，下一个请求只有在当前请求收到响应之后才会被发出。由于受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。流水线是在同一条长连接上连续发出请求，而不用等待响应返回，这样可以减少延迟
<img src="..\笔记图片\HTTPConnections.png" alt="HTTPConnections" style="zoom: 67%;" />

