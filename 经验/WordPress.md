# 搭建个人博客

1. **需要一台服务器**   ~~什么云都可以~~

2. 使用**docker**创建一个容器并**安装宝塔**

    1. 个人认为使用centos安装宝塔兼容性好一些所以选择centos系统

    2. 要用到的docker命令

        ```c++
        // 1. 拉取一个镜像
        docker pull centos 
        // 2. 开启并直接运行, -p 设置开启端口号(需要在防火墙上放行), --name 就是设置容器的名称
        docker run -p 8888:8888 -p 888:888 -p 443:443 -p 80:80 -p 20:20 -p 21:21 --name baota -itd centos
        // 3. 进入容器
        docker attach baota
        // 4. 挂起容器
        先按Ctrl + p, 然后 ctrl + q
        不要直接ctrl + d退出, 这样容器会直接关闭.当然重新再start也是一样的
        ```

    3. 进入容器安装宝塔

        ```tex
        一行超级方便
        
        yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
        ```

    4. 在容器内部输出bt, 得到下面画面说明成功

        ​	![显示如下](http://121.4.254.252/wp-content/uploads/2022/03/5ec9ad82420c818b731dfd8b7b57d04.png)

3. 宝塔面板可视化搭建WordPress

    1. 添加站点

        1. 域名没有可以直接填入IP地址
        2. php使用8.0, mysql5.7

            ![创建站点](http://121.4.254.252/wp-content/uploads/2022/03/6059be85fa15d234f0edb352a69099e.png)

    
    ​    
    
    2. 进入对应文件远程下载WordPress安装包, 并解压到当前目录
    
        > 下载地址: https://cn.wordpress.org/latest-zh_CN.zip
    
        ![解压](http://121.4.254.252/wp-content/uploads/2022/03/8f723193029736264cc90a19e7cba36.png)
    
        
    
    3. 将wordpress目录中的文件剪切到根目录, 最终文件结构如下
    
        ![文件结构](http://121.4.254.252/wp-content/uploads/2022/03/0aedf3cd08111344edb9f84f870d391.png)
    
    4. 访问 **ip地址/wp-admin/setup-config.php**, 如下所示即成功
    
        ![success](http://121.4.254.252/wp-content/uploads/2022/03/vnmry.jpg)

> **参考链接**
>
> [WordPress教程](https://www.easywpbook.com/)
>
> [宝塔安装](https://www.bt.cn/bbs/thread-19376-1-1.html)
>
> [docker安装宝塔](https://cloud.tencent.com/developer/article/1830313)
>
> [PHP点击出现下载页面的处理办法](https://blog.csdn.net/tangbin0505/article/details/103333004?spm=1001.2101.3001.6650.9&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-9.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-9.pc_relevant_default&utm_relevant_index=12)
>
> [个人博客地址](http://121.4.254.252/)