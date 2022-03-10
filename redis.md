# springboot中使用redis



1. 导入依赖

    ```java
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    
    <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    </dependency>
    ```

2. 链接Redis

    ```yaml
    spring:
      redis:
        host: 127.0.0.0 #链接远程服务器上的Redis
        port: 6379
    ```
    
3. 配置RedisConfig, 使用CacheManager

    ```java
    @Configuration
    public class RedisConfig {
    
        /**
         * cacheManager
         * @param factory
         * @return
         */
        @Bean
        public CacheManager cacheManager(RedisConnectionFactory factory) {
            // 生成两套默认配置，通过 Config 对象即可对缓存进行自定义配置
            RedisCacheConfiguration cacheConfig1 = RedisCacheConfiguration.defaultCacheConfig()
                    // 设置过期时间 10 分钟
                    .entryTtl(Duration.ofMinutes(10))
                    // 设置缓存前缀
                    .prefixKeysWith("cache:user:")
                    // 禁止缓存 null 值
                    .disableCachingNullValues()
                    // 设置 key 序列化
                    .serializeKeysWith(keyPair())
                    // 设置 value 序列化
                    .serializeValuesWith(valuePair());
            RedisCacheConfiguration cacheConfig2 = RedisCacheConfiguration.defaultCacheConfig()
                    // 设置过期时间 30 秒
                    .entryTtl(Duration.ofSeconds(30))
                    .prefixKeysWith("cache:user_info:")
                    .disableCachingNullValues()
                    .serializeKeysWith(keyPair())
                    .serializeValuesWith(valuePair());
            // 返回 Redis 缓存管理器
            return RedisCacheManager.builder(factory)
                    .withCacheConfiguration("user", cacheConfig1)
                    .withCacheConfiguration("userInfo", cacheConfig2)
                    .build();
        }
    
        /**
         * 配置键序列化
         * @return StringRedisSerializer
         */
        private RedisSerializationContext.SerializationPair<String> keyPair() {
            return RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer());
        }
    
        /**
         * 配置值序列化，使用 GenericJackson2JsonRedisSerializer 替换默认序列化
         * @return GenericJackson2JsonRedisSerializer
         */
        private RedisSerializationContext.SerializationPair<Object> valuePair() {
            return RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer());
        }
    
    }
    
    ```

4. 增删改查

    ```java
    @Service
    @CacheConfig(cacheNames = "userInfo")
    public class UserInfoServiceImpl implements UserInfoService {
        private static Map<String, UserInfo> userInfoMap;
    	//模拟数据库
        static {
            userInfoMap = new HashMap<>();
            userInfoMap.put("lele",new UserInfo("lele","male", 12));
            userInfoMap.put("sasa",new UserInfo("sasa","male", 12));
            userInfoMap.put("ewew",new UserInfo("ewew","male", 12));
            userInfoMap.put("aass",new UserInfo("aass","male", 12));
            userInfoMap.put("qqqw",new UserInfo("qqqw","male", 12));
        }
        @Override
        public void addUserInfo(UserInfo userInfo) {
            userInfoMap.put(userInfo.getName(), userInfo);
        }
    
        @Override
        @Cacheable(key = "#name", unless = "#result==null")
        public UserInfo getByName(String name) {
            if (!userInfoMap.containsKey(name)) {
                return null;
            }
            return userInfoMap.get(name);
        }
    
        @Override
        @CachePut(key = "#userInfo.name")
        public UserInfo updateUserInfo(UserInfo userInfo) {
    
            if (!userInfoMap.containsKey(userInfo.getName())) {
                throw new RuntimeException("该用户信息没有找到");
            }
    
            // 获取存储的对象
            UserInfo newUserInfo = userInfoMap.get(userInfo.getName());
            // 复制要更新的数据到新对象，因为不能更改用户名信息，所以忽略
            BeanUtils.copyProperties(userInfo, newUserInfo, "name");
            // 将新的对象存储，更新旧对象信息
            userInfoMap.put(newUserInfo.getName(), newUserInfo);
            // 返回新对象信息
            return newUserInfo;
        }
    
        @Override
        @CacheEvict(key = "#name")
        public void deleteByName(String name) {
            userInfoMap.remove(name);
        }
    }
    ```

     1. @CacheConfig

        ```yaml
        放在类前, 配置共有属性
        ```

     2. @CacheAble

        ```java
        可用于类或方法上；在目标方法执行前，会根据key先去缓存中查询看是否有数据，有就直接返回缓存中的key对应的value值。不再执行目标方法；无则执行目标方法，并将方法的返回值作为value，并以键值对的形式存入缓存
            
        public class Cache(){
            @CacheAble(cacheNames = "xx", key = "xx")
            public String add(User user) {
                return "success";
            }
        }
        ```

     3. @CachePut

        ```java
        可用于类或方法上；在执行完目标方法后，并将方法的返回值作为value，并以键值对的形式存入缓存中.
            
        public class Cache(){
            @CachePut(cacheNames = "xx", key = "xx", unless="#result==null")
            public String update(String username) {
                return "success";
            }
        }
        ```

        

     4. @CacheEvict

        ```java
        可用于类或方法上；在执行完目标方法后，清除缓存中对应key的数据(如果缓存中有对应key的数据缓存的话)
        
        public class Cache(){
            @CacheEvict(cacheNames = "xx", key = "xx")
            public String delete(String username) {
                return "success";
            }
        }
        ```

5. 开启缓存@EnableCaching

    ```java
    @SpringBootApplication
    @EnableCaching
    public class SpringbootFrameTestApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(SpringbootFrameTestApplication.class, args);
        }
    }
    ```

    

