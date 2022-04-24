

## 服务器初始化

### 配置免密登录

```tex
1. 编写.ssh/config文件
Host aly_server
	HostName ip地址
	User username
	Port 20000 (22可以不写)
2. ssh aly_server,  填写密码
3. ssh-copy-id aly_server
4. ssh aly_server, 不用填密码
```

### 添加用户

```
ubuntu: adduser
添加sudu权限: sudo usermod -G sudo username 
添加root权限:  sudo vim /etc/sudoers

修改文件
# User privilege specification
root ALL=(ALL) ALL
username ALL=(ALL) ALL
保存退出，username 用户就拥有了root权限。
```

### 安装Docker

[Docker安装](https://www.runoob.com/docker/ubuntu-docker-install.html)

`为docker添加sudo权限: sudo usermod -aG docker $USER`

