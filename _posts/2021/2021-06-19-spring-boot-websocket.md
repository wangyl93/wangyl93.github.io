---
layout: post
title: Spring-Boot快速集成WebSocket服务端 客户端(客户端消息同步回调)
category: springboot
no-post-nav: true
tags: [springboot]
keywords: Spring Boot,Websocket
excerpt: Spring Boot 和 Websocket
---

## Websocket 介绍

WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向

数据传输。

![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cucnVub29iLmNvbS93cC1jb250ZW50L3VwbG9hZHMvMjAxNi8wMy93cy5wbmc?x-oss-process=image/format,png#pic_center)

在 WebSocket API 中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。

## WebSocket 服务端实现

**使用Spring-Boot实现webSocket倒是简单，基于注解开发真的太爽了。**

*WebSocket服务端 pom.xml 依赖*

``` xml
<!-- WebSocket服务端依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

如果不是Sping-Boot框架可能换成其他的socket.api的依赖

我这里定义两个类，一个配置类配置socket的参数，一个WebSocketServer的实现类，使用时会注入IOC容器。

*WebSocket 服务端配置类*

```java
/**
 * WebSocket 服务端配置类
 */
@Configuration
public class WebSocketServerConfig {

    /**
     * ServerEndpointExporter bean 注入
     * @return
     */
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        ServerEndpointExporter serverEndpointExporter = new ServerEndpointExporter();
        return serverEndpointExporter;
    }

}
```

这里其他配置我暂时未配置，但是实际使用时，要配置一些参数优化服务。**ServerEndpointExporter**是必须要注入的，后续服务端类的实装需要用到。

*WebSocket服务端*

``` java
/**
 *
 * WebSocket服务端
 * @author DavidLei
 *
 */
@Component
@ServerEndpoint("/webSocket/{clientId}")
public class CustomizedWebSocketServer {

    /**
     * 日志
     */
    private Logger logger = LoggerFactory.getLogger(CustomizedWebSocketServer.class);

    /**
     * 在线数
     */
    private static int onlineCount = 0;

    /**
     * 线程安全的存储连接session的Map
     */
    private static Map<String, CustomizedWebSocketServer> clients = new ConcurrentHashMap<String, CustomizedWebSocketServer>();

    /**
     * session
     */
    private Session session;

    /**
     * 客户端端标识
     */
    private String clientId;

    /**
     * 客户端连接时方法
     * @param clientId
     * @param session
     * @throws IOException
     */
    @OnOpen
    public void onOpen(@PathParam("clientId") String clientId, Session session) throws IOException {
        logger.info("onOpen: has new client connect -"+clientId);
        //
        this.clientId = clientId;
        this.session = session;
        addOnlineCount();
        clients.put(clientId, this);
        logger.info("onOpen: now has "+onlineCount+" client online");
    }

    /**
     * 客户端断开连接时方法
     * @throws IOException
     */
    @OnClose
    public void onClose() throws IOException {
        logger.info("onClose: has new client close connection -"+clientId);
        clients.remove(clientId);
        subOnlineCount();
        logger.info("onClose: now has "+onlineCount+" client online");
    }

    /**
     * 收到消息时
     * @param message
     * @throws IOException
     */
    @OnMessage
    public void onMessage(String message) throws IOException {
        logger.info("onMessage: [clientId: " + clientId + " ,message:" + message + "]");
    }

    /**
     * 发生error时
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        logger.info("onError: [clientId: " + clientId + " ,error:" + error.getCause() + "]");
    }

    /**
     * 指定端末发送消息
     * @param message
     * @param clientId
     * @throws IOException
     */
    public void sendMessageByClientId(String message, String clientId) throws IOException {
        for (CustomizedWebSocketServer item : clients.values()) {
            if (item.clientId.equals(clientId) ) {
                item.session.getAsyncRemote().sendText(message);
            }
        }
    }

    /**
     * 所有端末发送消息
     * @param message
     * @throws IOException
     */
    public void sendMessageAll(String message) throws IOException {
        for (CustomizedWebSocketServer item : clients.values()) {
            item.session.getAsyncRemote().sendText(message);
        }
    }

    public static synchronized int getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        CustomizedWebSocketServer.onlineCount++;
    }

    public static synchronized void subOnlineCount() {
        CustomizedWebSocketServer.onlineCount--;
    }

    public static synchronized Map<String, CustomizedWebSocketServer> getClients() {
        return clients;
    }
}
```

注意@S**erverEndpoint**("/webSocket/{clientId}")注解，这里标识的时Socket的地址，以我的配置为例
我的tomcat端口是**8082**,**socket**的链接地址就是 **ws://localhost:8082/webSocket/12345678**

我因为业务需要我这里实现了两个方法 **sendMessageByClientId** **sendMessageAll**
一个是向连接到服务端的指定客户端发送信息，一个是向所有在线客户端发送信息。

## WebSocket客户端实现

*WebSocket客户端 pom.xml 依赖*

``` java
<!--WebSocket客户端 核心依赖包-->
<dependency>
    <groupId>org.java-websocket</groupId>
    <artifactId>Java-WebSocket</artifactId>
    <version>1.3.8</version>
 </dependency>
```

## *定义WebSocket客户端*

``` java
/**
 * 自定义WebSocket客户端
 */
public class CustomizedWebSocketClient extends WebSocketClient {

    /**
     * 日志
     */
    private Logger logger = LoggerFactory.getLogger(CustomizedWebSocketClient.class);

    /**
     * 线程安全的Boolean -是否受到消息
     */
    public AtomicBoolean hasMessage = new AtomicBoolean(false);

    /**
     * 线程安全的Boolean -是否已经连接
     */
    private AtomicBoolean hasConnection = new AtomicBoolean(false);

    /**
     * 构造方法
     *
     * @param serverUri
     */
    public CustomizedWebSocketClient(URI serverUri) {
        super(serverUri);
        logger.info("CustomizeWebSocketClient init:" + serverUri.toString());
    }

    /**
     * 打开连接是方法
     *
     * @param serverHandshake
     */
    @Override
    public void onOpen(ServerHandshake serverHandshake) {
        logger.info("CustomizeWebSocketClient onOpen");
    }

    /**
     * 收到消息时
     *
     * @param s
     */
    @Override
    public void onMessage(String s) {
        hasMessage.set(true);
        logger.info("CustomizeWebSocketClient onMessage:" + s);
    }

    /**
     * 当连接关闭时
     *
     * @param i
     * @param s
     * @param b
     */
    @Override
    public void onClose(int i, String s, boolean b) {
        this.hasConnection.set(false);
        this.hasMessage.set(false);
        logger.info("CustomizeWebSocketClient onClose:" + s);
    }

    /**
     * 发生error时
     *
     * @param e
     */
    @Override
    public void onError(Exception e) {
        logger.info("CustomizeWebSocketClient onError:" + e);
    }

    @Override
    public void connect() {
        if(!this.hasConnection.get()){
            super.connect();
            hasConnection.set(true);
        }
    }
}
```

*WebSocket客户端配置类*

```java
/**
 * WebSocket客户端配置类
 */
@Configuration
public class WebSocketClientConfig {

    /**
     * socket连接地址
     */
    @Value("${com.dl.socket.url}")
    private String webSocketUri;

    /**
     * 注入Socket客户端
     * @return
     */
    @Bean
    public CustomizedWebSocketClient initWebSocketClient(){
        URI uri = null;
        try {
            uri = new URI(webSocketUri);
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
        CustomizedWebSocketClient webSocketClient = new CustomizedWebSocketClient(uri);
        //启动时创建客户端连接
         webSocketClient.connect();
        return webSocketClient;
    }

}
```

*webSocketUri 的yml配置*

```yml
#socket客户端连接地址
com.dl.socket.url: ws://localhost:8082/webSocket//12345678
```

到目前为止正常的需求实装完了

*实装测试api*

```java
@Controller
@RequestMapping(value = "/socket")
public class WebSocketController {

    @Autowired
    private CustomizedWebSocketServer websocketServerCustomized;

    @Autowired
    private CustomizedWebSocketClient socketClient;

    @ResponseBody
   @PostMapping(value = "/message")
    public void getSocketMessage(HttpServletRequest request) throws IOException {
       JSONObject json = new JSONObject();
       json.put("to", request.getSession().getId());
       json.put("msg", "欢迎连接WebSocket！！！！");
       websocketServerCustomized.sendMessageAll(json.toJSONString());
    }
 }
```

*客户端后台log*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223436963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMjcxNTYx,size_16,color_FFFFFF,t_70)

*客户端后台log*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223551780.png)

正常的收发是完全ok的

## 客户端消息同步回调

但是这远远不够，我还有其他的业务需求，现在需要调用第三方的socket接口来实时反馈数据，但是第三方的接口后台有着很多运算，往往反馈数据很慢。但是前台调用我的api时有需要返回这些反馈数据。所以需要接口同步回调数据。

*加强版的客户端*

```java
/**
 * 自定义WebSocket客户端
 */
public class CustomizedWebSocketClient extends WebSocketClient {

    /**
     * 日志
     */
    private Logger logger = LoggerFactory.getLogger(CustomizedWebSocketClient.class);

    /**
     * 消息回调接口
     */
    private WebSocketClientSyncCallback callbck = null;

    /**
     * 线程安全的Boolean -是否受到消息
     */
    private AtomicBoolean hasMessage = new AtomicBoolean(false);

    /**
     * 线程安全的Boolean -是否已经连接
     */
    private AtomicBoolean hasConnection = new AtomicBoolean(false);

    /**
     * 构造方法
     *
     * @param serverUri
     */
    public CustomizedWebSocketClient(URI serverUri) {
        super(serverUri);
        logger.info("CustomizeWebSocketClient init:" + serverUri.toString());
    }

    /**
     * 打开连接是方法
     *
     * @param serverHandshake
     */
    @Override
    public void onOpen(ServerHandshake serverHandshake) {
        logger.info("CustomizeWebSocketClient onOpen");
    }

    /**
     * 收到消息时
     *
     * @param s
     */
    @Override
    public void onMessage(String s) {
        hasMessage.set(true);
        if(callbck!=null) {
            callbck.callback(s);
        }
        logger.info("CustomizeWebSocketClient onMessage:" + s);
    }

    /**
     * 当连接关闭时
     *
     * @param i
     * @param s
     * @param b
     */
    @Override
    public void onClose(int i, String s, boolean b) {
        this.hasConnection.set(false);
        this.hasMessage.set(false);
        logger.info("CustomizeWebSocketClient onClose:" + s);
    }

    /**
     * 发生error时
     *
     * @param e
     */
    @Override
    public void onError(Exception e) {
        logger.info("CustomizeWebSocketClient onError:" + e);
    }


    /**
     * 带有回调的消息发送接口
     * @param text
     * @param callback
     * @throws NotYetConnectedException
     */
    public void send(String text, WebSocketClientSyncCallback callback) throws NotYetConnectedException {
        logger.info("CustomizeWebSocketClient send:" + text);
        hasMessage.set(false);
        //设定回调接口
        this.callbck = callback;
        super.send(text);
        //计算等待；10s返回消息 超过10s直接退出
        for (int count = 0; ; ) {
            logger.debug("socketClient wait:"+count+" second, hasMessage："+hasMessage);
            //判断是否收到消息||socket返回数据超时
            if (hasMessage.get()||count>10) {
                break;
            } else if (count <=10) {
                try {
                   TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                count++;
            }
        }
    }

    @Override
    public void connect() {
        if(!this.hasConnection.get()){
            super.connect();
            hasConnection.set(true);
        }
    }
}
```

这里我重载了**send**方法，可以传入回调接口。同时还加上了线程等待，注意这里的判断参数，要换成线程安全的**AtomicBoolean**
当触发onMessage方法时置成true，默认为false，客户端**send**时置成false。

*socket客户端消息同步回调接口*

```java
/**
 * socket客户端消息同步回调接口
 */
public interface WebSocketClientSyncCallback {

    /**
     * socket客户端消息回调
     * @param message
     */
    void callback(String message);

}
```

*实装测试api*

```java
    @ResponseBody
    @PostMapping(value = "/clientCallback")
    public String testClientCallback(HttpServletRequest request)  {

        try {
            socketClient.send("test message",new WebSocketClientSyncCallback(){
                @Override
                public void callback(String message) {
                    callbackMessage=message;
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }

        return  callbackMessage;
    }
```

*socket服务端后台log*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501225534684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMjcxNTYx,size_16,color_FFFFFF,t_70)

*客户端后台log*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501225228124.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMjcxNTYx,size_16,color_FFFFFF,t_70)

可见后台socket客户端等待了3s后收到服务端反馈的数据，实现了同步回调。