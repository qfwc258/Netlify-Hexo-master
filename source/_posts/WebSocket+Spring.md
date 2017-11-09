title: WebSocket + Spring的使用
date: 2017-11-4 11:30:45
comments: true
tag:
 - WebSocket
 - Spring
categories: 后端技术

----------

## 应用场景
websocket 是 Html5 新增加特性之一，目的是浏览器与服务端建立全双工的通信方式，解决 http 请求-响应带来过多的资源消耗，同时对特殊场景应用提供了全新的实现方式，比如聊天、股票交易、游戏等对对实时性要求较高的行业领域。

## 使用方法

### 1. 使用WebSocket底层API
WebSocket 是发送和接收消息的底层API，WebSocket 协议提供了通过一个套接字实现全双工通信的功能。也能够实现 web 浏览器和 server 间的异步通信，全双工意味着 server 与浏览器间可以发送和接收消息。需要注意的是必须考虑浏览器是否支持，浏览器的支持情况如下：

![](http://oih7sazbd.bkt.clouddn.com/1.png)

<!-- more -->
#### 1.1 写Handler处理类

**方法1：实现 WebSocketHandler 接口，WebSocketHandler 接口如下**
```java
public interface WebSocketHandler {
	void afterConnectionEstablished(WebSocketSession session) throws Exception;
	void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception;
	void handleTransportError(WebSocketSession session, Throwable exception) throws Exception; 
	void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception; 
	boolean supportsPartialMessages();
}
```

**方法2：扩展 AbstractWebSocketHandler（更加简单一点）**
```java
@Service
public class ChatHandler extends AbstractWebSocketHandler {
	private static final Logger logger = LoggerFactory.getLogger(ChatHandler.class);
	private final static List<WebSocketSession> sessions = Collections.synchronizedList(new ArrayList<WebSocketSession>());
	// handle text msg.
	@Override
	protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
		session.sendMessage(new TextMessage("hello world."));
	}
}
```
除此之外,还可以重载其他三个方法
```java
handleBinaryMessage()
handlePongMessage()
handleTextMessage()
```

**方法3： 扩展 TextWebSocketHandler**

TextWebSocketHandler 是文本 WebSocket 处理器，继承 AbstractWebSocketHandler

你可能对连接的建立和关闭感兴趣，可以重载 afterConnectionEstablished() and afterConnectionClosed()方法:

```java
// 当新连接建立的时候，被调用;
public void afterConnectionEstablished(WebSocketSession session) throws Exception {
	logger.info("Connection established");
	logger.info("getId:" + session.getId());
	logger.info ("getLocalAddress:" + session.getLocalAddress().toString());
	logger.info ("getUri:" + session.getUri().toString());
}
// 当连接关闭时被调用；
@Override
public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
	logger.info("Connection closed. Status: " + status);
}
```

#### 1.2 拦截器实现
```java
package com.websocket;
@Service//如果采用WebSocketConfig类配置，需加上这一行；采用XML配置则不用
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        if (request instanceof ServletServerHttpRequest) {
            attributes.put("username",userName);
        }
        return true;
    }
    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {
    }
}
```

对以上代码分析：
> beforeHandshake()方法，在调用 handler 前调用。常用来注册用户信息，绑定 WebSocketSession，在 handler 里根据用户信息获取WebSocketSession发送消息。

#### 1.3 Spring MVC配置
**方法1：编写 WebSocketConfig 配置类，实现 WebSocketConfigurer 接口。**
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer{
	@Autowired
	private ChatHandler chatHandler;
	@Autowired
	private WebSocketHandshakeInterceptor webSocketHandshakeInterceptor;
	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(chatHandler,"/chat").addInterceptors(webSocketHandshakeInterceptor);
	}
}
```

对以上代码分析：
> * 实现 WebSocketConfigurer 接口，重写 registerWebSocketHandlers 方法，这是一个核心实现方法，配置 websocket 入口，允许访问的域、注册 Handler、SockJs 支持和拦截器。

> * registry.addHandler()注册和路由的功能，当客户端发起 websocket 连接，把 /path 交给对应的 handler 处理，而不实现具体的业务逻辑，可以理解为收集和任务分发中心。

> * addInterceptors，顾名思义就是为 handler 添加拦截器，可以在调用 handler 前后加入我们自己的逻辑代码。
方法二：通过 XML 配置文件添加配置

**方法2：通过 XML 配置文件添加配置**
```
<bean id="chatHandler" class="com.websocket.ChatHandler"/>
<websocket:handlers>
    <!-- 配置消息处理bean和路径的映射关系 -->
    <websocket:mapping path="/chat" handler="chatHandler"/>
    <!-- 配置握手拦截器 -->
    <websocket:handshake-interceptors>
        <bean class="com.websocket.WebSocketHandshakeInterceptor"/>
    </websocket:handshake-interceptors>
    <!-- 开启sockjs，去掉则关闭sockjs -->
    <websocket:sockjs/>
</websocket:handlers>
<!-- 配置websocket消息的最大缓冲区长度，可以不配 -->
<bean class="org.springframework.web.socket.server.standard.ServletServerContainerFactoryBean">
    <property name="maxTextMessageBufferSize" value="8192"/>
    <property name="maxBinaryMessageBufferSize" value="8192"/>
</bean>
```

#### 1.4 客户端配置
```javascript
var wsServer = 'ws://127.0.0.1:8080/chat'; 
var websocket = new WebSocket(wsServer); 
websocket.onopen = function (evt) { onOpen(evt) }; 
websocket.onclose = function (evt) { onClose(evt) }; 
websocket.onmessage = function (evt) { onMessage(evt) }; 
websocket.onerror = function (evt) { onError(evt) }; 
function onOpen(evt) { 
   console.log("Connected to WebSocket server."); 
} 
function onClose(evt) { 
   console.log("Disconnected"); 
} 
function onMessage(evt) { 
   console.log('Retrieved data from server: ' + evt.data); 
} 
function onError(evt) { 
   console.log('Error occured: ' + evt.data); 
}
websocket.send(“test”);//客户端向服务器发送消息
```

**注意：**

其中 **wsServer = ‘ws://127.0.0.1:8080/chat’** 中的地址要根据自己的实际情况来定，一般形式为：**ws://域名:端口/应用路径/WebSocket**

“应用路径”是应用部署在 tomcat 中的文件夹路径，“WebSocket 配置的 path”是配置文件中这条配置项配置的路径。

```
- Connection established
- getId:1
- getLocalAddress:/127.0.0.1:8080
- getUri:/chat
```

### 2. 使用SockJs
SockJS 是在 WebSocket 之上的 API，应对许多浏览器不支持 WebSocket 协议

#### 2.1 Spring MVC配置
添加 withSockJS()方法
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer{
	@Autowired
	private ChatHandler chatHandler;
	@Autowired
	private WebSocketHandshakeInterceptor webSocketHandshakeInterceptor;
	@Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
      //允许连接的域,只能以http或https开头
      String[] allowsOrigins = {"https://www.lyu3.com"};  
      //WebSocket通道,withSockJS()表示开启sockjs
      registry.addHandler(chatHandler,"/chat").setAllowedOrigins(allowsOrigins).addInterceptors(webSocketHandshakeInterceptor).withSockJS();
	}
}
```
对以上代码分析：
> * 和利用 websocket API 不同的是加上了 SockJs 支持。
> * setAllowedOrigins(String[] domains),允许指定的域名或 IP (含端口号)建立长连接，如果只允许自家域名访问，这里轻松设置。如果不限时使用 * 号，如果指定了域名，则必须要以 http 或 https 开头。

#### 2.2 客户端配置
具体做法是 依赖于 JavaScript 模块加载器（如 require.js or curl.js）　还是简单使用 script 标签加载 JavaScript 库。最简单的方法是 使用 script 标签从 SockJS CDN 中进行加载，如下所示：
```JavaScript
//加载sockjs
<script src="http://cdn.sockjs.org/sockjs-0.3.min.js"></script>
  var url = '/chat';
  var sock = new SockJS(url);
  //.....
```
对以上代码分析：
> * SockJS 所处理的 URL 是 “http://“ 或 “https://“ 模式，而不是 “ws://“ or “wss://“；
> * 其他的函数如 onopen, onmessage, and onclose ，SockJS 客户端与 WebSocket 一样，在此代码省略。

WebSocket服务器后台输出入如下：
```
- Connection established
- getId: qtfwdtti**（注意：此处和直接利用websocket API有区别）**
- getLocalAddress:/127.0.0.1:8080
- getUri: /chat/668/lyusantu/websocket**（注意：此处和直接利用websocket API有区别）**
```

### 利用STOMP
SockJS 为 WebSocket 提供了 备选方案。但无论哪种场景，对于实际应用来说，这种通信形式层级过低。下面看一下如何 在 WebSocket 之上使用 STOMP协议，来为浏览器 和 server 间的 通信增加适当的消息语义。（STOMP—— Simple Text Oriented Message Protocol——面向消息的简单文本协议）

#### 3.1 STOMP 与 WebSocket 的关系
假设HTTP协议并不存在，只能使用TCP套接字来编写web应用，你可能认为这是一件疯狂的事情。不过 幸好，我们有 HTTP协议，它解决了 web 浏览器发起请求以及 web 服务器响应请求的细节。直接使用 WebSocket（SockJS） 就很类似于 使用 TCP 套接字来编写 web 应用；因为没有高层协议，因此就需要我们定义应用间所发送消息的语义，还需要确保 连接的两端都能遵循这些语义。

**同HTTP在TCP套接字上添加请求-响应模型层一样，STOMP在 WebSocket之上提供了一个基于帧的线路格式层，用来定义消息语义。**

#### 3.2 STOMP帧
STOMP帧由命令，一个或多个头信息以及负载所组成。如下就是发送数据的一个STOMP帧：
```
   SEND
destination:/app/room-message
content-length:20
{\"message\":\"Hello!\"}
```
对以上代码分析：
- SEND：STOMP命令，表明会发送一些内容
- destination：头信息，用来表示消息发送到哪里
- content-length：头信息，用来表示 负载内容的 大小；
- 空行
- 帧内容（负载）内容

#### 3.3 示例代码
##### 1. stomp节点配置

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.AbstractWebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
  
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {    
	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		//添加一个/chat端点，客户端就可以通过这个端点来进行连接；withSockJS作用是添加SockJS支持
		registry.addEndpoint(“/chat”).withSockJS();
	}
	@Override
	public void configureMessageBroker(MessageBrokerRegistry registry) {
		//定义了两个客户端订阅地址的前缀信息，也就是客户端接收服务端发送消息的前缀信息
		registry.enableSimpleBroker(“/message”,”/notice”);
		//定义了服务端接收地址的前缀，也即客户端给服务端发消息的地址前缀
		registry.setApplicationDestinationPrefixes(“/app”);
	}
}
```
对以上代码分析：
- EnableWebSocketMessageBroker 注解表明： 这个配置类不仅配置了 WebSocket，还配置了基于代理的 STOMP 消息

- 它复写了 registerStompEndpoints() 方法：添加一个服务端点，来接收客户端的连接。将 “/chat” 路径注册为 STOMP 端点。这个路径与之前发送和接收消息的目的路径有所不同， 这是一个端点，客户端在订阅或发布消息到目的地址前，要连接该端点，即用户发送请求 ：url=’/127.0.0.1:8080/chat’ 与 STOMP server 进行连接，之后再转发到订阅url

- 它复写了 configureMessageBroker() 方法：配置了一个 简单的消息代理，通俗一点讲就是设置消息连接请求的各种规范信息

- 发送应用程序的消息将会带有 “/app” 前缀

##### 2. controller实现
```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.stereotype.Controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import com.app.deploying.model.Message;
import com.spring.pojo.Greeting;
import com.spring.pojo.HelloMessage;
@Controller
public class GreetingController {
	@Autowired
	private SimpMessagingTemplate simpMessagingTemplate;
	
	@MessageMapping("/room-message/{roomId}")
	public void roomMessage(Message message, @DestinationVariable String roomId){
		String englishName = message.getTo();
		String dest = "/message/" + roomId + "/" + englishName;
		LogConstant.runLog.info(roomId+"号房间——"+"收到"+message.getFrom()+"发送至"+englishName+"的消息："+"\""+message.getContent()+"\"");
		message.setDate(new Timestamp(System.currentTimeMillis()));
		this.simpMessagingTemplate.convertAndSend(dest, message);
	}
}
```
对以上代码分析：
- @MessageMapping 标识客户端发来消息的请求地址，前面我们全局配置中制定了服务端接收的地址以“/app”开头，所以客户端发送消息的请求连接是：/app/room-message/ + 房间id

- @DestinationVariable 用以取出请求地址中的房间 id 参数 roomId；

- this.simpMessagingTemplate.convertAndSend(dest, message) 这个方法官方给出的解释是 Convert the given Object to
 serialized form, possibly using a MessageConverter, wrap it as a message and send it to the given destination. 意思就是“将给定的对象进行序列化，使用 ‘MessageConverter’ 进行包装转化成一条消息，发送到指定的目标”，通俗点讲就是我们使用这个方法进行消息的转发发送

##### 3. 客户端实现
首先引用sockjs.js 和 stomp.js
```java
//连接服务器的函数
function connect() {
    var socket = new SockJS('/socket');
    stompClient = Stomp.over(socket);
    stompClient.connect('', '', function (frame) {
    	console.log('Connected: ' + frame);
    	setConnected(true);
    	var messageModel = {};
    	messageModel.type = 1;
    	messageModel.content = encodeURIComponent(nickName+","+englishName);
		//向服务端发送消息
    	stompClient.send("/app/room-notice/"+roomId, {}, JSON.stringify(messageModel));
		//订阅服务器发送来的消息
    	stompClient.subscribe('/message/'+roomId+"/"+englishName, function (event) {
    		consoleMessage(event.body);
    	});
	}, function () {
        connect();
    });
}
//发送聊天信息
function sendMessage() {
	var messageModel = {};
	messageModel.type = 2;
	var content = $('#chat').val();
	var toUserName = $("#toUserName").val();
	messageModel.content = encodeURIComponent(content);
	messageModel.from = englishName;
	messageModel.to = toUserName;
    stompClient.send("/app/room-message/"+roomId, {}, JSON.stringify(messageModel));
    $('#chat').val("");
}
//显示聊天信息
function consoleMessage(message){
	var msgJson = JSON.parse(message);
	var type = msgJson.type;
	var content = decodeURIComponent(msgJson.content);//得到消息内容
	//。。。。。
}
```
对以上代码分析：
- 利用 stompClient 的 connect(login, passcode, connectCallback, errorCallback, vhost) 方法建立连接，值得注意的是不同版本的 stomp.js 的 connect() 函数的参数会有所不同

- encodeURIComponent 方法是把发送内容作为URI组件进行编码，可以防止中文乱码问题

- 利用 stompClient 的 send(destination, headers, body) 方法可以向服务端发送消息，第一个参数为发送消息地址，最后一个参数是发送消息的 json 串

- 利用 stompClient 的 subscribe(destination, callback, headers) 方法可以订阅服务器发送来的消息，destination 表示服务器发送消息地址；通过 event 的 body 获取消息内容

- decodeURIComponent对encodeURIComponent() 函数编码的内容进行解码，得到消息内容