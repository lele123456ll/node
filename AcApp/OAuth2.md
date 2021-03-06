# OAuth2学习

## 一、使用实例

>  	在一款游戏中， 需要获得AcWing用户的头像与昵称信息。这样就需要获得用户的授权才可以得到用户信息， AcWing端才会同意读取信息。**传统方法为： 直接将AcWing端账户密码告诉游戏， 既可以读取到用户信。**

### **缺点**

1. `游戏会保存用户的账号与密码, 这很不安全`
2. `AcWing不得不部署密码登录，而单纯的密码登录并不安全。`
3. `游戏会获得用户在AcWing端的所有信息, 用户无法限制其授权范围以及有效期`
4. `用户要想收回自己的"权力", 只有去修改密码, 这样所有获得用户授权的第三方登录都会失效`
5. `只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏`

### **OAuth就是为了解决上面的问题而诞生的**

## 二、名词解释

```c++
1. Third-party application：第三方应用程序
2. HTTP service：HTTP服务提供商
3. Resource Owner：资源所有者
4. User Agent：用户代理
5. Authorization server：认证服务器，即服务提供商专门用来处理认证的服务器。
6. Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。
```

## 三、OAuth的思路

```c++
OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。

"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。
```

## 四、运行流程

- 图解

    ![OAuth2](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/oauth2.png)

## 五、客户端授权模式

- OAuth2定义了**四种方式**

1. 授权码模式（authorization code）[AcWing使用](D:\md-node\typora\AcApp\AcApp-Oauth2.md)
2. 简化模式（implicit）
3. 密码模式（resource owner password credentials）
4. 客户端模式（client credentials）

### 1. 授权码模式

- 图解

    ![授权码模式](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E6%8E%88%E6%9D%83%E7%A0%81%E6%A8%A1%E5%BC%8F.png)

- 流程

    ```c++
    （A）用户访问客户端，后者将前者导向认证服务器。
    
    （B）用户选择是否给予客户端授权。
    
    （C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
    
    （D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
    
    （E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。
    
    ```

### 2. 简化模式

- 图解

    ![简化模式](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E7%AE%80%E5%8C%96%E6%A8%A1%E5%BC%8F.png)

- 流程

    ```c++
    （A）客户端将用户导向认证服务器。
    
    （B）用户决定是否给于客户端授权。
    
    （C）假设用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"，并在URI的Hash部分包含了访问令牌。
    
    （D）浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值。
    
    （E）资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。
    
    （F）浏览器执行上一步获得的脚本，提取出令牌。
    
    （G）浏览器将令牌发给客户端。
    ```

### 3. 密码模式

- 图解

    ![密码模式](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E5%AF%86%E7%A0%81%E6%A8%A1%E5%BC%8F.png)

- 流程

    ```c++
    （A）用户向客户端提供用户名和密码。
    
    （B）客户端将用户名和密码发给认证服务器，向后者请求令牌。
    
    （C）认证服务器确认无误后，向客户端提供访问令牌。
    ```

### 4. 客户端模式

- 图解

    ![客户端模式](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%A8%A1%E5%BC%8F.png)

- 流程

    ```
    （A）客户端向认证服务器进行身份认证，并要求一个访问令牌。
    
    （B）认证服务器确认无误后，向客户端提供访问令牌。
    ```

    

