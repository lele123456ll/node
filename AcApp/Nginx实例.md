# [Nginx快速学习部署](https://www.acwing.com/blog/content/13954/)

## 核心文件

```
http {
     server {
         listen ;
         server_name ;
         location 路径 {
                proxy_pass
                }
     }
       server {
         listen ;
         server_name ;
         location 路径 {
                proxy_pass
                }
     }
       server {
         listen ;
         server_name ;
         location 路径 {
                proxy_pass
                }
     }
  ....
}
```

- server各个字段含义

    ```
    listen : 要监听的端口号
    server_name : 解析哪个域名发来的请求
    location : 转发地址
    ```

    

    