# ElasticSearch

## **什么是Elasticsearch？**

> Elasticsearch 是一个**实时**的**分布式存储、搜索、分析**的引擎。

## **为什么要用Elasticsearch**

> Elasticsearch的强大之处就是可以**模糊查询**。

```sql
虽然可以通过如下暴力的搜索也可以进行查询, 但是当数据量很大时(亿级), 查询时间就会是秒级,这可太慢了!

select * from user where name like '%xxx%'
```

> ElasticSearch是专门用来做搜索的

## ElasticSearch的数据结构

>  一些常见的数据结构对应的时间复杂度
>
> - 树形:log(n) 
> - 链表:O(n)
> - hash表:O(1)

1. Elasticsearch会进行**分词**。中文使用**IK分词器**

    ![IK](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/uof7z5nfd4.jpeg)

2. 真实数据结构如下

    ![数据结构](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/ElasticSearch%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpeg)

    ```
    Term Dictionary : 分词器分词后的所有词的集合
    我们不可能去遍历整个集合寻找我们需要的值, 所以通过FST的形式得到Term Index
    FST:
    1. 空间占用小。通过对词典中单词前缀和后缀的重复利用，压缩了存储空间；
    2. 查询速度快。O(len(str))的查询时间复杂度。
    
    Term Index : 这层只存储部分词的前缀，存在内存中
    PostingList : 词对应的页码, 通过Frame Of Reference（FOR）编码技术对里边的数据进行压缩，节约磁盘空间。
    ```

    1. [FST](https://www.cnblogs.com/jiu0821/p/7688669.html)
    2. **[对PostingList处理详解](https://zhuanlan.zhihu.com/p/33671444)**

    

## ElasticSearch中的术语

- **Index**：Elasticsearch的Index相当于数据库的Table
- **Type**：这个在新的Elasticsearch版本已经废除（在以前的Elasticsearch版本，一个Index下支持多个Type--有点类似于[消息队列](https://cloud.tencent.com/product/cmq?from=10680)一个topic下多个group的概念）
- **Document**：Document相当于数据库的一行记录
- **Field**：相当于数据库的Column的概念
- **Mapping**：相当于数据库的Schema的概念
- **DSL**：相当于数据库的SQL（给我们读取Elasticsearch数据的API）

## 项目中有关ES的技术

1. MySql与ES的同步 -> **logstash**

    1. 在config中配置mysql.conf

    ```
    input{
        jdbc{
            # 要使用的驱动包类
            jdbc_driver_class => "com.mysql.jdbc.Driver"
            # mysql数据库的连接信息
            jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/blog"
            # mysql用户
            jdbc_user => "root"
            # mysql密码
            jdbc_password => "123456"
            # 定时任务，多久执行一次查询，默认一分钟，如果想要没有延迟，可以使用 schedule => "* * * * * *"
            schedule => "* * * * *"
            # 清空上传的sql_last_value记录
            clean_run => true
            # 你要执行的语句
            statement => "select * FROM t_blog WHERE update_time > :sql_last_value AND update_time < NOW() ORDER BY update_time desc"  
        }
    }
    
    output {
        elasticsearch{
            # es host : port
            hosts => ["127.0.0.1:9200"]
            # 索引
            index => "blog"
            # _id
            document_id => "%{id}"
        }
    }
    ```

2. 模糊查询

    1. **原始ES模糊查询语句**

        ```json
        POST /blog/_search 
        {
          "query": {
            "bool": {
              "should": [
                {
                  "match_phrase": {
                    "title": "springboot"
                  }
                },
                {
                  "match_phrase": {
                    "content": "springboot"
                  }
                }
              ]
            }
          }
        }
        ```

    2. **SpringBoot对应的查询语句**

        ```
        // 获取QueryBuilders中bool查询对象
        BoolQueryBuilder builder = QueryBuilders.boolQuery();
        
        // 模糊查询
        builder.should(QueryBuilders.matchPhraseQuery("title",param.getKeyword()));
        builder.should(QueryBuilders.matchPhraseQuery("content",param.getKeyword()));
        
        //使用esBlogRepository的search方法查询
        Page<EsBlog> search =(Page<EsBlog>)esBlogRepository.search(builder);
            
        List<EsBlog> content = search.getContent();
        ```







