# Nginx

## 是什么

> Nginx是一款自由的、开源的、高性能的HTTP服务器和反向代理服务器；同时也是一个IMAP、POP3、SMTP代理服务器；Nginx可以作为一个HTTP服务器进行网站的发布处理，另外Nginx可以作为反向代理进行负载均衡的实现。

## 用途

###	正向代理

- 好处

    ```c++
    （1）访问原来无法访问的资源，如Google
    （2） 可以做缓存，加速访问资源
    （3）对客户端访问授权，上网进行认证
    （4）代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息
    ```

- 图解

    ![正向代理](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E6%AD%A3%E5%90%91%E4%BB%A3%E7%90%86.png)

### 反向代理

- 好处

    ```
    （1）保证内网的安全，通常将反向代理作为公网访问地址，Web服务器是内网
    （2）负载均衡，通过反向代理服务器来优化网站的负载
    ```

- 图解

    ![反向代理](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86.jpg)

### 区别

- 图解

    <img src="https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/5434ce3b590d86068091c034b7e53f1b6629f62b.png" alt="区别" style="zoom:80%;" />

- 分析

    ```
    在正向代理中，Proxy和Client同属于一个LAN（图中方框内），隐藏了客户端信息；
    在反向代理中，Proxy和Server同属于一个LAN（图中方框内），隐藏了服务端信息；
    ```

    

