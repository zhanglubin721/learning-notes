## Spring提供了处理网络请求的能力

- @Controller
- @RestController
- @RequestMapping, @GetMapping, @PostMapping 等
- @ResponseBody, @RequestBody
- DispatcherServlet（Spring MVC 的核心）
- Spring 的 IoC 容器、AOP、事务管理等

🔹 这些注解属于 **Spring MVC 模块**，是 Spring Framework 提供的。

你即使不使用 Spring Boot，纯用 Spring Framework 配上 web.xml + DispatcherServlet 也能用这些注解构建 Web 应用。

## Spring Boot内嵌Tomcat

Spring Boot 主要目的是为了简化 Spring 应用开发，它**不改变 Spring 本身的特性**，但提供了以下便利：

- **内嵌 Tomcat、Jetty、Undertow**（默认是 Tomcat）

  ➤ 这是 Spring Boot 提供的能力，通过 starter 模块（如 spring-boot-starter-web）自动配置嵌入式 Servlet 容器。

- **自动配置（AutoConfiguration）**

  ➤ Spring Boot 会自动配置 DispatcherServlet、视图解析器等，无需手动配置。

- **@SpringBootApplication 注解**

  ➤ 它是 @Configuration + @EnableAutoConfiguration + @ComponentScan 的组合。

- **application.properties / application.yml 配置简化**

## 三次握手

```
浏览器（客户端）              Spring Boot 应用（内嵌 Tomcat，监听8080）
     |                               |
     | ------ SYN -----------------> |   第一次握手：客户端发起连接请求
     | <----- SYN + ACK ------------ |   第二次握手：服务端同意连接
     | ------ ACK -----------------> |   第三次握手：客户端确认连接
     |         ✅连接建立成功         |
```

## 建立连接

```java
while (running) {
    SocketChannel socket = serverSocketChannel.accept();
    poller.register(socket); // 注册到 Poller 中
}
```

每次执行 serverSocketChannel.accept() 就是**接收一个客户端 TCP 连接**，也就是**建立一个 SocketChannel 实例**，代表这条连接。

- ServerSocketChannel 是服务端监听的 socket；
- accept() 方法会阻塞或非阻塞（取决于配置），直到有客户端连接进来；
- 每次 accept() 返回的 SocketChannel 对象，就对应一个客户端；
- 接收到这个连接之后，Tomcat 把它**交给 Poller 注册事件监听**（比如是否有数据可读）；

**所以：**

- **一个连接 = 一个 SocketChannel**；
- 这个 while 循环每次处理一个连接（即一个客户端）；
- 有成百上千个客户端连接时，Tomcat 会多线程处理，每个连接最终会被 Poller、Worker 线程协作处理数据。

## 处理HTTP请求全流程

当然可以，下面我将以 **一条 HTTP 请求从到达 Tomcat 到被 Controller 处理** 的全过程，用**纯文字**详细、清晰地串联起来，帮你建立一整条完整的链路理解。

### **整体流程概览：**

**ServerSocketChannel → SocketChannel → Poller → 读取 HTTP 报文 → 封装 Request/Response → Spring 处理请求**

### **步骤一：ServerSocketChannel 监听端口**

- 当 Tomcat 启动时，它创建了一个 ServerSocketChannel 并将其绑定到端口（比如 8080）；
- 这个 ServerSocketChannel 就是监听客户端请求的“服务器门口”；
- 它处于“接收连接”的模式，就像在门口等着访客进来。

### **步骤二：客户端发起连接，SocketChannel 建立**

- 浏览器或 Postman 发出 HTTP 请求，底层是 TCP 协议；
- 操作系统层完成 **三次握手**，Tomcat 通过 accept() 方法接收这个连接；
- 一旦连接建立，Tomcat 拿到一个 SocketChannel，代表和这个客户端的一对一通信通道。

### **步骤三：注册 SocketChannel 到 Poller**

- Tomcat 不直接读取数据，而是将这个 SocketChannel 注册到一个 Selector（多路复用器）中；
- Poller 是一个内部线程，不断轮询 Selector，查看这些连接是否“有数据可读”；
- 相当于一个保安巡视连接们，看谁要“说话”（发来 HTTP 报文）了。

### **步骤四：Poller 检测到可读事件，触发读取**

- 当浏览器真的发出完整 HTTP 报文（如 GET /hello）；
- Poller 检测到这个连接的 SocketChannel 可读；
- Tomcat 分派一个 SocketProcessor 来读取数据。

### **步骤五：读取 HTTP 报文并解析**

- SocketProcessor 使用 Http11Processor 来处理这个连接；
- 它从 SocketChannel 中读取字节流；
- 然后解析成 HTTP 协议结构：请求行（GET /xxx）、请求头、请求体；
- 解析完成后，Tomcat 构造出：
  - org.apache.catalina.connector.Request（封装请求）
  - org.apache.catalina.connector.Response（封装响应）

### **步骤六：将 Request/Response 交给 Servlet 容器处理**

- Tomcat 是 Servlet 容器，它会调用应用中注册的 Servlet；
- Spring Boot 中注册的是 DispatcherServlet；
- 所以 Tomcat 会执行：

```java
dispatcherServlet.service(request, response)
```

- 从这里开始，请求进入 Spring MVC 的世界。

### **步骤七：Spring 分发到 Controller**

- DispatcherServlet 根据请求路径找到对应的 @Controller 方法；
- 调用对应的方法，如：

```java
@GetMapping("/user/{id}")
public User getUser(@PathVariable Long id) { ... }
```

- 方法返回后，Spring 把返回值写入 response.getWriter()；
- 响应最终通过 SocketChannel 写回给浏览器；
- 一个请求-响应流程就此完成。

### **总结：关键组件一条链**

| **阶段** | **组件**            | **作用**                     |
| -------- | ------------------- | ---------------------------- |
| 1        | ServerSocketChannel | 监听端口，接收连接           |
| 2        | SocketChannel       | 表示一个 TCP 连接            |
| 3        | Poller + Selector   | 检测是否可读                 |
| 4        | Http11Processor     | 读取字节流并解析 HTTP        |
| 5        | Request/Response    | 封装请求响应对象             |
| 6        | DispatcherServlet   | 分发请求到 Spring Controller |
| 7        | Controller 方法     | 真正处理业务逻辑             |