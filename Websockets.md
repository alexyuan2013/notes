## 对WebSocket的学习与总结

### 背景

Web应用的构建思路很简单，即一个单向的信息交流：客户端向服务端发送请求，服务端向客户端返回请求的结果，客户端再渲染结果。
这种交流始于客户端请求，止于服务端返回结果。

这种模式最大的缺点是客户端发送请求后必须等待服务端的返回，才能执行下一步的操作。后来AJAX解决了这个问题，实现了异步请求和
页面的局部刷新。但是，这一切都是基于HTTP协议的，HTTP协议要带很多额外的信息(Header)，这对于一些高频变化的数据就显得不是
很高效。

WebSocket应运而生。WebSocket是HTML5开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
WebSocket通信协议于2011年被IETF定为标准RFC 6455，WebSocketAPI被W3C定为标准。
在WebSocket API中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。
两者之间就直接可以数据互相传送。[<sup>1</sup>][1]

WebSocket相对于HTTP协议而言，有更小的报文头，传输量更小，减小了Kb级的传输，延时也由150毫秒减少到50毫秒；
可以从服务端向客户端主动发送数据。
WebSocket首先通过HTTP建立连接，即握手，然后将HTTP连接更换成全双工的WebSocket连接，连接的任意一端都可以关闭连接。
客户端和服务端都可以通过这条连接发送信息，Websocket通过使用ping/pong心跳框架来实现持久连接。

和HTTP及HTTPS协议类似，WebSocket使用ws或wss的schemes，如ws://www.sample.org/或wss://www.sample.org/。

WebSocket最重要的应用场景还是面向的那些需要高频交流数据的场景，所谓的高频交流差不多每100毫秒一次。

WebSocket通过帧来传递数据，文本和二进制之间只有很小的差别，因此对于传送的消息内容而言是相对中立的。在传输消息时，
就需要消息带上元数据，即服务端和客户端都能接受的一个应用层的协议，这个协议被称为**子协议**。
选用哪个子协议在第一次握手的时候决定。WebSocket并没有强制使用哪种子协议，服务端和客户端就需要采用预定义的标准格式或者
自定义的格式。STOMP，即Simple Text Orientated Messaging Protocol，又称STOMP over WebSocket，就是WebSocket传输
的一个子协议。


### Spring与WebSocket

WebSocket的服务端的实现最流行可能不是Spring，不过由于第一个项目原因，还是采用了相对比较熟悉的Spring来实现。
从Spring 4开始加入了对WebSocket的支持，其将诸多概念，如消息传递（messaging）、通道（channel）、处理器（handler）等，
集成到了消息映射（message mapping）的注解（annotation）中。你可以使用Spring创建一个简单的不基于任何子协议的WebSocket
对话，那么你要自己处理消息传递的格式；也可以使用STOMP向所有客户端广播消息或仅仅向某个客户端发送消息。
接下来就一次介绍这三种方式的实现：

#### 简单的WebSocket应用

该应用并没有使用WebSocket的子协议，服务端和客户端需要自己定义消息的格式。
采用的是Spring Boot的形式，避免了xml文件的配置。
服务端需要编写一个处理器（endpoint）来接受、提取客户端的消息并返回相应的信息，使用Spring，你可以从TextWebSocketHandler
或BinaryWebSocketHandler两个类中扩展出自己的Handler，前者用来处理文本，后者用来处理二进制，即图片或其他多媒体文件。
这里以处理文本为例，代码如下：

```java
public class SampleTextWebSocketHandler extends TextWebSocketHandler {
  @Override
  protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
    String payload = message.getPayload();
    JSONObject jsonObject = new JSONObject(payload);
    StringBuilder builder = new StringBuilder();
    builder.append("From Myserver-").append("Your Message: ").append(jsonObject.get("clientMessage"));
    session.sendMessage(new TextMessage(builder.toString()));
  }
}
```
handleTextMessage方法包含了获取客户端的消息（`message.getPayload()`）并将其转化为JSON数据的方法，
最后将相关信息发送给客户端（`session.sendMessage(new TextMessage(builder.toString()))`）。
为了告诉Spring客户端的请求端点(handler)，需要对处理器进行注册：

```java
@Configuration
@EnableWebSocket
public class SampleEhoWebSocketConfigure {
  @Bean
  WebSocketConfigure webSocketConfigure (final WebSocketHandler webSocketHandler) {
    return new WebSocketConfigure() {
      @Override
      public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new SampleTextWebSocketHandler(), "/path/wsAddress");
      }
    };
  }
  
  @Bean
  WebSocketHandler myWebsocketHandler() {
    return SampleTextWebSocketHandler();
  }
}
```

注解@Configuration和@EnableWebSocket告诉Spring这是关于WebSocket的配置，其注册了先前的handler（SampleTextWebSocketHandler），
并且绑定了handler的请求地址。
现在的问题是Spring如何将所有的东西集成到一起？Spring Boot提供了一种非常简单的方式，其内嵌了web服务器，你只需运行即可。

```java
@SpringBootApplication
public class EchoWebSocketBootApplication {
  public static void main(String[] args) {
    SpringApplication.run(EchoWebSocketBootApplication.class, args);
  }
}
```
注解@SpringBootApplication将EchoWebSocketBootApplication标记为一个特殊的配置类，其作用与以下注解类似：

- @Configuration，声明应用上下文的bean定义

- @EnableAutoConfiguration，告诉Spring根据classpath添加一个bean的定义（例如，classpath中的spring-webmvc包，
告诉Spring Boot启动一个web应用，并使用web.xml文件中的注册的DispatcherServerlet）

- @ComponentScan， 扫描同一个包中的所有注解（services, controller, configuration等等）

这样就没有使用任何的xml的配置文件。

当客户端需要发送WebSocket请求时，应该创建一个JavaScript对象（`ws = new WebSocket('ws://localhost:8090/path/wsAddress')`），
同时，为了能接收数据，需要实现一个回调函数的监听（`ws.onmessage`）和错误处理的监听（`ws.onerror`），代码如下：

```javascript
function openWebSocket(){
  ws = new WebSocket('ws://localhost:8090/path/wsAddress');
  ws.onmessage = function(event){
    renderServerReturnData(event.data);
  };
  ws.onerror = function(event){
    $('#errDiv').html(event);
  };
}

function sendMyClientMessage() {
  var myText = document.getElementById('myText').value;
  var message = JSON.stringify({'clientName': 'Client-' + randomnumber, 'clientMessage': myText});
  ws.send(message);
  document.getElementById('myText').value='';
}
```
至此，服务端和客户端的代码都已经完成，在Chrome或IE10上运行时，一切正常，但是在IE10一下版本时，就会报错。
Spring提供了不支持WebSocket的解决方法，下边将介绍。

#### 发布-订阅模式





#### 发布-订阅模式——发送消息到单个用户




[1]: https://zh.wikipedia.org/wiki/WebSocket "维基百科"
