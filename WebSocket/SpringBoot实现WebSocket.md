# WebSocket

## 一、为什么需要WebSocket

> HTTP协议有一个缺陷， 只能由客户端发起。
>
> **HTTP 协议做不到服务器主动向客户端推送信息。**

## 二、简介

> 它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息。

- 图解

    ![http&ws](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/http&ws.png)

- 特点

    ```java
    （1）建立在 TCP 协议之上，服务器端的实现比较容易。
    
    （2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
    
    （3）数据格式比较轻量，性能开销小，通信高效。
    
    （4）可以发送文本，也可以发送二进制数据。
    
    （5）没有同源限制，客户端可以与任意服务器通信。
    
    （6）协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。
    ```

## 三、客户端

### 实例

```javascript
let websocket = null;

// 开启
websocket.onopen() = function() {
    // todo
}
// 关闭
websocket.onclose() = function() {
    // todo
}
// 接收消息
websocket.onmessage() = function() {
    //todo
}
// 发送消息
websocket.send(message);
```

### [具体API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

## 四、SpringBoot服务端

### 原生注解

- 注解

    **@ServerEndpoint**
    	通过这个 spring boot 就可以知道你暴露出去的 ws 应用的路径，有点类似我们经常用的@RequestMapping。比如你的启动端口是 8080，而这个注解的值是 ws，那我们就可以通过 ws://127.0.0.1:8080/ws 来连接你的应用	
    **@OnOpen**
    当 websocket 建立连接成功后会触发这个注解修饰的方法，注意它有一个  Session 参数

    **@OnClose**
    当 websocket 建立的连接断开后会触发这个注解修饰的方法，注意它有一个  Session 参数

    **@OnMessage**

    当客户端发送消息到服务端时，会触发这个注解修改的方法，它有一个 String 入参表明客户端传入的值

    **@OnError**

    当 websocket 建立连接时出现异常会触发这个注解修饰的方法，注意它有一个  Session 参数	

- config类

    ```java
    @Configuration
    public class WebSocketConfig {
        @Bean
        public ServerEndpointExporter serverEndpointExporter(){
            return new ServerEndpointExporter();
        }
    }
    ```

- websocket类

    ```java
    @ServerEndpoint("/websocket")
    @Component
    @Slf4j
    public class WebSocketTest {
    
        private static int onlineCount = 0;
        private static CopyOnWriteArrayList<WebSocketTest> webSocketSet = new CopyOnWriteArrayList<WebSocketTest>();
        private Session session;
    
        @OnOpen
        public void onOpen(Session session) {
            this.session = session;
    
            webSocketSet.add(this);
            addOnlineCount();
            log.info("新连接加入, 当前连接人数为 : " + getOnlineCount());
        }
        @OnClose
        public void onClose() {
            webSocketSet.remove(this);
            subOnlineCount();
            log.info("断开一人, 当前连接人数为 : " + getOnlineCount());
        }
        @OnMessage
        public void onMessage(String message, Session session) {
            log.info("来自客户端消息 : " + message);
    
            // 群发
            for (WebSocketTest item : webSocketSet) {
                try {
                    item.sendMessage(message);
                } catch (IOException e) {
                    e.printStackTrace();
                    continue;
                }
            }
    
        }
        @OnError
        public void onError(Session session,Throwable throwable){
            log.error("发生错误");
            throwable.printStackTrace();
        }
    
    
        public void sendMessage(String message) throws IOException {
            this.session.getBasicRemote().sendText(message);
        }
    
        public static synchronized int getOnlineCount() {
            return onlineCount;
        }
    
        public static synchronized void addOnlineCount() {
            WebSocketTest.onlineCount++;
        }
    
        public static synchronized void subOnlineCount() {
            WebSocketTest.onlineCount--;
        }
    }
    ```

### Spring

> 1. 编写websocket处理类， HttpAuthHandler继承TextWebSocketHandler
> 2. 编写session管理类,，方便管理session。
> 3. 实现拦截器， 需要token才放行
> 4. 编写config类

- **HttpAuthHandler**

    - 继承TextWebSocketHandler类

    - 重写方法

        ```java
            /**
             * 建立连接  {@link javax.websocket.OnOpen}
             * @param session 会话
             * @throws Exception
             */
            @Override
            public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        
                Object token = session.getAttributes().get("token");
                if (token != null) {
                    // 用户连接成功，放入在线用户缓存
                    WsSessionManager.add(token.toString(), session);
                } else {
                    throw new RuntimeException("用户登录已经失效!");
                }
            }
        
         	/**
             * 处理信息 {@link javax.websocket.OnMessage}
             * @param session
             * @param message
             * @throws Exception
             */
            @Override
            protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
                // 获得客户端传来的消息
                String payload = message.getPayload();
                Object token = session.getAttributes().get("token");
        
                log.info("server 接收到 " + token + " 发送的 " + payload);
        
                log.info("向组内用户发送信息");
                Collection<WebSocketSession> sessions = WsSessionManager.getAll();
                for (WebSocketSession s : sessions) {
                    s.sendMessage(new TextMessage(payload));
                    log.info("session : " + s);
                }
            }
        
        	/**
             * 关闭连接  {@link javax.websocket.OnClose}
             * @param session
             * @param status
             * @throws Exception
             */
            @Override
            public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        
                Object token = session.getAttributes().get("token");
                if (token != null) {
                    // 用户退出，移除缓存
                    WsSessionManager.remove(token.toString());
                }
            }
        ```

- **WsSessionManager**

    - session池

        ```java
        public class WsSessionManager {
        
            /**
             * session池, 存放session
             */
            private static ConcurrentHashMap<String, WebSocketSession> SESSION_POOL = new ConcurrentHashMap<>();
        
            /**
             * 添加session
             * @param key
             * @param session
             */
            public static void add(String key, WebSocketSession session) {
                SESSION_POOL.put(key, session);
            }
        
            /**
             * 删除 session,会返回删除的 session
             * @param key
             * @return
             */
            public static WebSocketSession remove(String key) {
                return SESSION_POOL.remove(key);
            }
        
            /**
             * 移除并关闭session连接
             * @param key
             */
            public static void removeAndClose(String key) {
                WebSocketSession session = remove(key);
        
                if (session != null) {
                    try {
                        // 关闭session
                        session.close();
                    } catch (IOException e) {
                        // 异常
                        e.printStackTrace();
                    }
                }
            }
        
            /**
             * 获得session
             * @param key
             * @return
             */
            public static WebSocketSession get(String key) {
                return SESSION_POOL.get(key);
            }
        
            public static Collection<WebSocketSession> getAll() {
                return SESSION_POOL.values();
            }
        
        ```

- MyInterceptor

    - 实现HandshakeInterceptor

        ```java
         @Override
            public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        
                System.out.println("握手开始");
                // 获得请求参数, 在访问时加上token参数
                Map<String, String> paramMap = HttpUtil.decodeParamMap(request.getURI().getQuery(),StandardCharsets.UTF_8);
        
                String uid = paramMap.get("token");
        
                if (StrUtil.isNotBlank(uid)) {
                    // 放入属性域
                    attributes.put("token", uid);
                    System.out.println("用户 token " + uid + " 握手成功！");
                    return true;
                }
                System.out.println("用户登录已失效");
                return false;
            }
        
            @Override
            public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {
                System.out.println("握手完成");
            }
        ```

        

- WebSocketConfig

    - 实现WebSocketConfigurer,

    - **@EnableWebSocket开启**

        ```java
        @Configuration
        @EnableWebSocket
        public class WebSocketConfig implements WebSocketConfigurer {
            @Autowired
            private HttpAuthHandler httpAuthHandler;
            @Autowired
            private MyInterceptor myInterceptor;
        
            @Override
            public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
                registry
                        .addHandler(httpAuthHandler, "websocket")
                        .addInterceptors(myInterceptor)
                        .setAllowedOrigins("*");
            }
        }
        ```

        

### TIO

- **原生回调接口支持**

    ```java
    // 通过包扫描，反射创建新实例，赋给 TioWebSocketServerBootstrap中的属性
    
    //最主要的逻辑处理类，必须要写，否则 抛异常
    public class MyWebSocketMsgHandler  implements IWsMsgHandler {}
    //可不写
    public class SpringBootAioListener extends WsServerAioListener {}
    //可不写
    public class SpringBootGroupListener implements GroupListener {}
    //可不写
    public class SpringBootIpStatListener implements IpStatListener {}
    ```

- 导入依赖

    ```
    <dependency>
        <groupId>org.t-io</groupId>
        <artifactId>tio-websocket-spring-boot-starter</artifactId>
        <version>3.5.5.v20191010-RELEASE</version>
    </dependency>
    ```

- **@EnableTioWebSocketServer开启TIO**

- 编写yml配置

    ```yml
    # 自动配置类 : TioWebSocketServerAutoConfiguration
    # 如果想配置其他东西可以在里面找
    
    tio:
      websocket:
        server:
          port: 8082  #  连接端口号
          heartbeatTimeout: 30000 # 超时断开设置为30s, 方便测试
    ```

- **实现IWsMsgHandler**

    ```java
    @Component
    @Slf4j
    public class MyHandler implements IWsMsgHandler {
        private static final String GROUP_ID = "LELE";
    
        /**
         * 握手
         * @param httpRequest
         * @param httpResponse
         * @param channelContext
         * @return httpResponse
         * @throws Exception
         */
        @Override
        public HttpResponse handshake(HttpRequest httpRequest, HttpResponse httpResponse, ChannelContext channelContext) throws Exception {
            log.info("握手");
            String uuid = httpRequest.getParam("uuid");
    
            // 绑定用户
            Tio.bindUser(channelContext, uuid);
    
            return httpResponse;
        }
    
    
        /**
         * 握手成功
         * @param httpRequest
         * @param httpResponse
         * @param channelContext
         * @throws Exception
         */
        @Override
        public void onAfterHandshaked(HttpRequest httpRequest, HttpResponse httpResponse, ChannelContext channelContext) throws Exception {
            log.info("握手成功");
    
            // 绑定所属组
            Tio.bindGroup(channelContext, GROUP_ID);
            // 获得组内成员个数
            int count = Tio.getAll(channelContext.tioConfig).getObj().size();
            // 消息
            String msg = "{name:'admin',message:'" + channelContext.userid + " 进来了，共【" + count + "】人在线" + "'}";
            // 用tio-websocket，服务器发送到客户端的Packet都是WsResponse
            WsResponse wsResponse = WsResponse.fromText(msg, "UTF-8");
    
            //群发
            Tio.sendToGroup(channelContext.tioConfig, GROUP_ID, wsResponse);
        }
    
        /**
         * 接收二进制文件
         * @param wsRequest
         * @param bytes
         * @param channelContext
         * @return
         * @throws Exception
         */
        @Override
        public Object onBytes(WsRequest wsRequest, byte[] bytes, ChannelContext channelContext) throws Exception {
            return null;
        }
    
        /**
         * 断开连接
         * @param wsRequest
         * @param bytes
         * @param channelContext
         * @return
         * @throws Exception
         */
        @Override
        public Object onClose(WsRequest wsRequest, byte[] bytes, ChannelContext channelContext) throws Exception {
            log.info("断开连接");
            return null;
        }
    
        /**
         * 接收消息
         * @param wsRequest
         * @param msg
         * @param channelContext
         * @return
         * @throws Exception
         */
        @Override
        public Object onText(WsRequest wsRequest, String msg, ChannelContext channelContext) throws Exception {
            log.info("接收文本消息 : " + msg);
    
            // 用tio-websocket，服务器发送到客户端的Packet都是WsResponse
            WsResponse wsResponse = WsResponse.fromText(msg, "UTF-8");
    
            Tio.sendToGroup(channelContext.tioConfig, GROUP_ID, wsResponse);
    
            return null;
        }
    }
    ```

### STOMP 

