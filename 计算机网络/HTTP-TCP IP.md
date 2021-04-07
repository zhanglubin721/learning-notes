# HTTP请求

## HTTP请求的完全过程

### 1.1 浏览器根据域名解析IP地址

浏览器根据访问的域名找到其IP地址。DNS查找过程如下：

1. 浏览器缓存：首先搜索浏览器自身的DNS缓存（缓存的时间比较短，大概只有1分钟，且只能容纳1000条缓存），看自身的缓存中是否是有域名对应的条目，而且没有过期，如果有且没有过期则解析到此结束。
2. 系统缓存：如果浏览器自身的缓存里面没有找到对应的条目，那么浏览器会搜索操作系统自身的DNS缓存，如果找到且没有过期则停止搜索解析到此结束。
3. 路由器缓存：如果系统缓存也没有找到，则会向路由器发送查询请求。
   ISP（互联网服务提供商） DNS缓存：如果在路由缓存也没找到，最后要查的就是ISP缓存DNS的服务器。

### 1.2 浏览器与WEB服务器建立一个TCP连接

​    TCP的3次握手。

### 1.3 浏览器给WEB服务器发送一个HTTP请求

​    一个HTTP请求报文由请求行（request line）、请求头部（headers）、空行（blank line）和请求数据（request body）4个部分组成。

![](D:\learning-notes\计算机网络\image\20190527111928530.png)

**1.3.1 请求行**

​    请求行分为三个部分：请求方法、请求地址URL和HTTP协议版本，它们之间用空格分割。例如，GET /index.html HTTP/1.1。

**3.协议版本**

​    协议版本的格式为：HTTP/主版本号.次版本号，常用的有HTTP/1.0和HTTP/1.1

**1.3.2 请求头部**

​    请求头部为请求报文添加了一些附加信息，由“名/值”对组成，每行一对，名和值之间使用冒号分隔。

​    请求头部的最后会有一个空行，表示请求头部结束，接下来为请求数据。

![](D:\learning-notes\计算机网络\image\2019052711205938.png)

下面是一个POST方法的请求报文：

POST 　/index.php　HTTP/1.1 　　 请求行

Host: localhost

User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:10.0.2) Gecko/20100101 Firefox/10.0.2　　请求头

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,/;q=0.8

Accept-Language: zh-cn,zh;q=0.5

Accept-Encoding: gzip, deflate

Connection: keep-alive

Referer: http://localhost/

Content-Length：25

Content-Type：application/x-www-form-urlencoded

　　空行

username=aa&password=1234　　请求数据

### 1.4 服务器端响应HTTP请求，浏览器得到HTML代码

 HTTP响应报文由状态行（status line）、相应头部（headers）、空行（blank line）和响应数据（response body）4个部分组成。

![](D:\learning-notes\计算机网络\image\2019052711212421.png)

**1.4.1 状态行**

​    状态行由3部分组成，分别为：协议版本、状态码、状态码扫描。其中协议版本与请求报文一致，状态码描述是对状态码的简单描述。

![](D:\learning-notes\计算机网络\image\20190527112145931.png)

**1.4.2 响应头部**

![](D:\learning-notes\计算机网络\image\20190527112207584.png)

1.4.3 响应数据

       用于存放需要返回给客户端的数据信息。

HTTP/1.1 200 OK　　状态行

Date: Sun, 17 Mar 2013 08:12:54 GMT　　响应头部

Server: Apache/2.2.8 (Win32) PHP/5.2.5

X-Powered-By: PHP/5.2.5

Set-Cookie: PHPSESSID=c0huq7pdkmm5gg6osoe3mgjmm3; path=/

Expires: Thu, 19 Nov 1981 08:52:00 GMT

Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0

Pragma: no-cache

Content-Length: 4393

Keep-Alive: timeout=5, max=100

Connection: Keep-Alive

Content-Type: text/html; charset=utf-8

　　空行

 响应数据

```html
<html>　　
	<head>
		<title>HTTP响应示例<title>
	</head>

	<body>
		Hello HTTP!
	</body>

</html>
```

### 1.5断开连接

四次挥手

## 细节详解

### TCP连接

![](D:\learning-notes\计算机网络\image\20181011152421951.png)

过程:   

1.客户端发送SYN（SEQ=x）报文给服务器端，进入SYN_SEND状态。

2.服务器端收到SYN报文，回应一个SYN （SEQ=y），ACK(ACK=x+1）报文，进入SYN_RECV状态。

3.客户端收到服务器端的SYN报文，回应一个ACK(ACK=y+1）报文，进入Established（建立）状态。

  

为什么要三次握手？

一次握手：客户端无法确认是否能够和服务器正常通信，却一直发连接请求是毫无意义的。

两次握手：1.客户端发一条连接请求给服务器，由于网络阻塞，服务器未收到。

2.客户端等了一段时间，服务器仍未回应它，于是再次发出连接请求，服务器收到请求并进行确认，TCP连接建立，开始通信，通信结束后释放连接。客户端进入CLOSED状态。

3.此时服务端收到失效的连接请求，并向客户端确认，但客户端已关闭，服务端将会为连接请求分配资源并且一直等待下去，浪费了服务端连接资源。

四次握手：三次握手已经能够建立连接，期间服务器有一次确认连接请求的操作就已足够，没必要进行多次确认，对于资源来说是一种浪费。

### 释放TCP连接

![](D:\learning-notes\计算机网络\image\20181011152405216.png)

过程：

1.数据发送完毕后，客户端发送释放连接请求（FIN=1,seq=u ）并进入FIN-WAIT-1状态。

2.服务器收到释放连接请求，做出应答（ ACK=1,seq=v,ack=u+1.），并进入CLOSE-WAIT状态。（此时客户端处于FIN-WAIT-2状态不发送只接收数据，此时仍在接收服务器传输的数据）

3.服务器发送完所有数据后发送释放连接请求（FIN=1,ACK=1,seq=w,ack=u+1），并进入LAST-ACK状态。

4.客户端收到释放连接请求后发送确认应答（ACK=1,seq=u+1,ack=w+1),并进入TIME-WAIT状态.该状态会持续2MSL时间（服务器收到应答会立即进入CLOSED状态），若该时间段内没有收到重发请求,就进入CLOSED状态。

 

A:为什么四次挥手？

建立连接时， 服务器收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。 而释放连接，服务器接收到客户端的FIN报文时，表示客户端不再发送但还能接收数据，此时服务器未必将全部数据都发送给了客户端，因此会先发送ACK报文进入CLOSE-WAIT状态，待发送完所有数据后，再发送FIN报文给客户端表示同意关闭连接（ACK和FIN一般都会分开发送)，从而导致多了一次。

B:为什么客户端发完第四次挥手后需要持续2MSL时间后才会关闭？

如果客户端发完第四次挥手立即关闭：

第一，若最后一个ACK报文丢失，服务器收不到客户端的应答，会再次发送一个释放连接请求，而此时客户端已经关闭，服务器会一直等待并发送请求。

第二，若在此次连接中出现“已经失效的连接请求报文段”，下次建立的TCP连接中就会出现旧连接的请求报文（若客户端发送完第四次挥手后，在2MSL时间后关闭，此段时间内可以使本次连接内产生的所有报文段从网络中消失，下次建立的TCP连接中就不会出现旧连接的请求报文）。

# TCP/IP协议