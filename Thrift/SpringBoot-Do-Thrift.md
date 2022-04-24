# SpringBoot实现Thrift

## 一、通过thrift文件生成Java源代码

### thrift文件

- 官方模板

    ```c++
    namespace cl shared
    namespace cpp shared
    namespace d share // "shared" would collide with the eponymous D keyword.
    namespace dart shared
    namespace java shared
    namespace perl shared
    namespace php shared
    namespace haxe shared
    namespace netstd shared
    
    
    struct SharedStruct {
      1: i32 key
      2: string value
    }
    
    service SharedService {
      SharedStruct getStruct(1: i32 key)
    }
    ```
    
- 实例

    ```c++
    //这里指明了代码生成之后,所处的文件路径
    namespace java com.lele.thrift.shriftcode
    //将thrift的数据类型格式转换为java习惯的格式
    typedef i16 short
    typedef i32 int
    typedef i64 long
    typedef string String
    typedef bool boolean
    
    //定义学生对象
    struct Student {
        //optional 可选 非必传
        1:optional String name,
        2:optional int age,
        3:optional String address
    }
    
    //定义数据异常
    exception DataException {
        
        1:optional int code,
        2:optional String message,
        3:optional String dateTime
    }
    
    //定义操作学生的服务
    service StudentService {
        //根据名称获取学生信息 返回一个学生对象  抛出DataException异常
        //required 必传项
        Student getStudentByName(1:required String name) throws (1:DataException dataException),
    
        //保存一个学生信息 无返回 抛出DataException异常
        void save(1:required Student student) throws (1:DataException dataException)
    }
    ```

### 导出生成Java代码

- thrift命令格式 :`thrift --gen <language> <Thrift filename>`

- 导出Java代码 :  `thrift --gen java xxx.thrift`

- 生成Java文件分别存放在client端与server端

## 二、Server端实现

### 贴上生成的Java代码到包下

### 编写service实现类

- **实现Service.Iface接口, 交给Spring管理**

    ```java
    @Service // spring管理
    @Slf4j
    public class StudentServiceImpl implements StudentService.Iface{
        @Override
        public Student getStudentByName(String name) throws DataException, TException {
            // todo
            return null;
        }
    
        @Override
        public void save(Student student) throws DataException, TException {
            // todo
        }
    }
    
    ```

### 编写ThriftServer类

- 配置文件添加端口和线程信息

    ```
    server:
      thrift:
        port: 8899 #监听的端口
        min-thread-pool: 100  #线程池最小线程数
        max-thread-pool: 1000 #线程池最大线程数
    ```

- ThriftServer类

    ```java
    public class ThriftServer {
        /**
         * 端口号
         * */
        @Value("${server.thrift.port}")
        private Integer port;
    
        /**
         * 线程池最小线程数
         */
        @Value("${server.thrift.min-thread-pool}")
        private Integer minThreadPool;
    
        /**
         * 线程池最大线程数
         */
        @Value("${server.thrift.max-thread-pool}")
        private Integer maxThreadPool;
    
        /**
         * 业务服务对象
         */
        @Autowired
        StudentServiceImpl service;
        
        /**
         * 开启服务
         */
        public void start() {
            try {
                //1. 非阻塞的Socket, 通讯
                TNonblockingServerSocket socket = new TNonblockingServerSocket(port);
    
                //2. 使用THsHaServer, 线程
               	//配置参数
                THsHaServer.Args arg = new THsHaServer.Args(socket).minWorkerThreads(minThreadPool).maxWorkerThreads(maxThreadPool);
                //3. Processor  处理业务
                StudentService.Processor<StudentServiceImpl> processor = new StudentService.Processor<>(service);
                
                // arg参数设置工厂
                // 协议工厂 TCompactProtocol压缩工厂  二进制压缩协议
                arg.protocolFactory(new TCompactProtocol.Factory());
    
                // 传输工厂
                arg.transportFactory(new TFramedTransport.Factory());
    
                // 处理器工厂
                arg.processorFactory(new TProcessorFactory(processor));
    
                //4. 服务
                TServer server = new THsHaServer(arg);
    
                System.out.println("shrift server started; port:" + port);
                // 启动server
                // 异步非阻塞的死循环
                server.serve();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```

### 启动服务

- 注入对象

    ```java
    @SpringBootApplication
    public class ThriftServerApplication {
        public static void main(String[] args) {
            SpringApplication.run(ThriftServerApplication.class, args);
        }
        /**
         * 向Spring注册一个Bean对象
         * initMethod = "start"  表示会执行名为start的方法
         * start方法执行之后，就会阻塞, 接受客户端的请求
         * @return thriftServer
         */
        @Bean(initMethod = "start")
        public ThriftServer init() {
            ThriftServer thriftServer = new ThriftServer();
            return thriftServer;
        }
    }
    ```

    

## 三、Client端单例实现

### 贴上生成的Java代码到包下

### 编写ThriftClient类

- 单例实现

    ```java
    public class ThriftClient{
        /**
        * 连接的server端的信息
        */
        @Setter
        private String host;
        @Setter
        private Integer port;
    
        /**
         * 网络传输
         */
        private TTransport tTransport;
    
        /**
         * 网络协议
         */
        private TProtocol tProtocol;
    
        /**
         * 服务
         */
        private StudentService.Client client;
    
        /**
         * 初始化
         */
        @SneakyThrows
        private void init() {
            tTransport = new TFramedTransport(new TSocket(host, port), 600);
    
            //协议对象 这里使用协议对象需要和服务器的一致
            tProtocol = new TCompactProtocol(tTransport);
            client = new StudentService.Client(tProtocol);
        }
    
        /**
         * 获取client
         * @return client
         */
        public StudentService.Client getService() {
            return client;
        }
    
        /**
         * 开启client服务
         * @throws TTransportException
         */ 
        public void open() throws TTransportException {
            if (null != tTransport && !tTransport.isOpen()) {
                tTransport.open();
            }
        }
    
        /**
         * 关闭client服务
         */
        public void close() {
            if (null != tTransport && tTransport.isOpen()) {
                tTransport.close();
            }
        }
    }
    ```

### 编写ThriftClientConfig

- **作用** : 将ThriftClient交给Spring管理

    ```java
    @Configuration
    public class ThriftClientConfig {
        /**
         * 服务端的地址
         */
        @Value("${server.thrift.host}")
        private String host;
        /**
         * 服务端的端口
         */
        @Value("${server.thrift.port}")
        private Integer port;
    
        /**
         * 初始化Bean的时候调用对象里面的init方法
         * @return
         */
        @Bean(initMethod = "init")
        // 默认单例模式, 请求实例化一个新的ThriftClient连接对象
        @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
        public ThriftClient init() {
            ThriftClient thriftClient = new ThriftClient();
            thriftClient.setHost(host);
            thriftClient.setPort(port);
            return thriftClient;
        }
    }
    ```


## 四、Client池

> **一个Service建立一个连接之后，放在一个池子里面，需要使用的时候就直接拿出来用，用完再还回去；如果一定时间内一直没有人使用，就把资源释放掉**

### 引入依赖

- common-pool

    ```xml
    <dependency>
        <groupId>commons-pool</groupId>
        <artifactId>commons-pool</artifactId>
    </dependency>
    ```

###  创建连接对象

- TTSocket

    ```java
    public class TTSocket {
        //thrift socket对象
        private TSocket tSocket;
    
        // 网络传输
        private TTransport tTransport;
    
        // 网络协议
        private TProtocol tProtocol;
    
        // 服务客户端对象
        private StudentService.Client client;
    
        /**
         * 构造方法初始化各个连接对象
         * @param host server的地址
         * @param port server的端口
         */
        @SneakyThrows
        public TTSocket(String host, Integer port) {
            tSocket = new TSocket(host, port);
            tTransport = new TFramedTransport(tSocket, 600);
    
            //协议对象 这里使用协议对象需要和服务器的一致
            tProtocol = new TCompactProtocol(tTransport);
            client = new StudentService.Client(tProtocol);
        }
    
        /**
         * 获取服务客户端对象
         * @return 服务端对象
         */
        public StudentService.Client getService() {
            return client;
        }
        /**
         * 打开通道
         * @throws TTransportException
         */
        public void open() throws TTransportException {
            if (null != tTransport && !tTransport.isOpen()) {
                tTransport.open();
            }
        }
        /**
         * 关闭通道
         */
        public void close() {
            if (null != tTransport && tTransport.isOpen()) {
                tTransport.close();
            }
        }
        /**
         * 判断通道是否是正常打开状态
         * @return
         */
        public boolean isOpen() {
                Socket socket = tSocket.getSocket();
                return socket.isConnected() && !socket.isClosed();
        }
    }
    ```
    

### ThriftClientConnectPoolFactory —— 工厂对象

- 使用GenericObjectPool类

    > GenericObjectPool 创建时，可以用有参构造函数进行初始化，通过**GenericObjectPoolConfig** 和**PooledObjectFactory**来进行参数的初始化和对象工厂类的引入。

    - 方法

        ```
        1. 获得池中对象
        pool.borrowObjct();
        2. 移除一个对象
        pool.invalidateObject(obj);
        3. 将获得的对象返回给连接池
        pool.returnObject(obj);
        ```

    - **配置信息**——GenericObjectPoolConfig

        ```
        1. 默认配置信息
        GenericObjectPool.config
        ```

    - **连接工厂**——PooledObjectFactory

        ```java
        class ConnectionFactory extends BasePoolableObjectFactory {
                //远端地址
                private String host;
        
                //端口
                private Integer port;
        
                /**
                 * 构造方法初始化地址及端口
                 * @param ip
                 * @param port
                 */
                public ConnectionFactory(String ip, int port) {
                    this.host = ip;
                    this.port = port;
                }
                /**
                 * 创建一个对象
                 * @return
                 * @throws Exception
                 */
                @Override
                public Object makeObject() throws Exception {
                    // 实例化一个自定义的一个thrift 对象
                    TTSocket ttSocket = new TTSocket(host, port);
                    // 打开通道
                    ttSocket.open();
                    return ttSocket;
                }
        
                /**
                 * 销毁对象
                 * @param obj
                 */
                @Override
                public void destroyObject(Object obj) {
                    try {
                        if (obj instanceof TTSocket) {
                            //尝试关闭连接
                            ((TTSocket) obj).close();
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
        
                /**
                 * 校验对象是否可用
                 * 通过 pool.setTestOnBorrow(boolean testOnBorrow)
                 * 设置为true这会在调用pool.borrowObject()获取对象之前调用这个方法用于校验对象是否可用
                 * @param obj 待校验的对象
                 * @return
                 */
                @Override
                public boolean validateObject(Object obj) {
                    if (obj instanceof TTSocket) {
                        TTSocket socket = ((TTSocket) obj);
                        if (!socket.isOpen()) {
                            return false;
                        }
                        return true;
                    }
                    return false;
                }
            }
        ```

- **实现**

    ```java
    public class ThriftClientConnectPoolFactory {
        /**
         * 对象池
         */
        public GenericObjectPool pool;
    
        /**
         * 实例化池工厂帮助类
         *
         * @param config
         * @param ip
         * @param port
         */
        public ThriftClientConnectPoolFactory(GenericObjectPool.Config config, String ip, int port) {
            ConnectionFactory factory = new ConnectionFactory(ip, port);
            //实例化池对象
            this.pool = new GenericObjectPool(factory, config);
            //设置获取对象前校验对象是否可以
            this.pool.setTestOnBorrow(true);
        }
    
        /**
         * 在池中获取一个空闲的对象
         * 如果没有空闲且池子没满，就会调用makeObject创建一个新的对象
         * 如果满了，就会阻塞等待，直到有空闲对象或者超时
         *
         * @return
         * @throws Exception
         */
        public TTSocket getConnect() throws Exception {
            return (TTSocket) pool.borrowObject();
        }
    
        /**
         * 将对象从池中移除
         *
         * @param ttSocket
         */
        public void invalidateObject(TTSocket ttSocket) {
            try {
                pool.invalidateObject(ttSocket);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    
        /**
         * 将一个用完的对象返还给对象池
         * @param ttSocket
         */
        public void returnConnection(TTSocket ttSocket) {
            try {
                pool.returnObject(ttSocket);
            } catch (Exception e) {
                if (ttSocket != null) {
                    try {
                        ttSocket.close();
                    } catch (Exception ex) {
                        log.error(ex.getMessage(), ex);
                    }
                }
                log.error(e.getMessage(), e);
            }
        }
    
        /**
         * 池里面保存的对象工厂
         */
        class ConnectionFactory extends BasePoolableObjectFactory {
            //远端地址
            private String host;
    
            //端口
            private Integer port;
    
            /**
             * 构造方法初始化地址及端口
             *
             * @param ip
             * @param port
             */
            public ConnectionFactory(String ip, int port) {
                this.host = ip;
                this.port = port;
            }
    
            /**
             * 创建一个对象
             * @return
             * @throws Exception
             */
            @Override
            public Object makeObject() throws Exception {
                // 实例化一个自定义的一个thrift 对象
                TTSocket ttSocket = new TTSocket(host, port);
                // 打开通道
                ttSocket.open();
                return ttSocket;
            }
    
            /**
             * 销毁对象
             * @param obj
             */
            @Override
            public void destroyObject(Object obj) {
                try {
                    if (obj instanceof TTSocket) {
                        //尝试关闭连接
                        ((TTSocket) obj).close();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
    
            /**
             * 校验对象是否可用
             * 通过 pool.setTestOnBorrow(boolean testOnBorrow)
             * 设置为true这会在调用pool.borrowObject()获取对象之前调用这个方法用于校验对象是否可用
             *
             * @param obj 待校验的对象
             * @return
             */
            @Override
            public boolean validateObject(Object obj) {
                if (obj instanceof TTSocket) {
                    TTSocket socket = ((TTSocket) obj);
                    if (!socket.isOpen()) {
                        return false;
                    }
                    return true;
                }
                return false;
            }
        }
    }
    
    ```

### Spring管理工厂对象

- 修改ThriftClientConfig

    ```java
    @Bean
    public ThriftClientConnectPoolFactory ThriftClientPool() {
        //对象池的配置对象, 默认配置
        GenericObjectPool.Config config = new GenericObjectPool.Config();
        //创建一个池工厂对象
        ThriftClientConnectPoolFactory thriftClientConnectPoolFactory = new ThriftClientConnectPoolFactory(config, host, port);
        //交由Spring管理
        return thriftClientConnectPoolFactory;
    }
    ```

### Service使用

- 接口

    ```java
    public interface StudentService {
        /**
         * 根据姓名获取学生信息
         * @param name 姓名
         * @return 学生
         */
        Student getStudentByName(String name);
    
        /**
         * 保存学生信息
         * @param student 学生
         */
        void save(Student student);
    }
    ```

- 实现

    > 1. 获取连接池对象
    > 2. 通过对象获取连接
    > 3. 通过连接获取服务并且调用方法
    > 4. 将连接返回到池中
    > 5. 出现异常移除连接

    ```java
    @Service
    @Slf4j
    public class StudentServiceImpl implements StudentService {
    
        //1. 获取连接池对象
        @Autowired
        ThriftClientConnectPoolFactory thriftClientPool;
    
        @Override
        public Student getStudentByName(String name) {
            TTSocket ttSocket = null;
            try {
                //2. 通过对象获取连接
                ttSocket = thriftClientPool.getConnect();
                log.info("ttSocket : "  +  ttSocket);
    
                log.info("客户端请求用户名为:" + name + "的数据");
    		   //3. 通过连接获取服务并且调用方法
                Student student = ttSocket.getService().getStudentByName(name);
    
                log.info("获取成功！！！服务端返回的对象:" + student);
    
                //4. 将连接返回到池中
                thriftClientPool.returnConnection(ttSocket);
                return student;
            } catch (Exception e) {
                e.printStackTrace();
                //5. 出现异常移除连接
                thriftClientPool.invalidateObject(ttSocket);
            }
            return null;
        }
    
        @Override
        public void save(Student student) {
            TTSocket ttSocket = null;
            try {
                ttSocket = thriftClientPool.getConnect();
                log.info("ttSocket : "  +  ttSocket);
    
                log.info("客户端请求保存对象:" + student);
                ttSocket.getService().save(student);
    
                log.info("保存成功");
                thriftClientPool.returnConnection(ttSocket);
    
            } catch (Exception e) {
                e.printStackTrace();
                thriftClientPool.returnConnection(ttSocket);
            }
        }
    }
    ```

## 五、结果

- 使用Postman进行测试

![res](http://121.4.254.252/wp-content/uploads/2022/03/thrift-res.png)
