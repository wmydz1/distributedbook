#选择合适的NoSQL存储系统

there are so many NoSQL database storage tools out there. It’s almost as bad as brands of sport drinks or water. Have you noticed that some mega-supermarkets have whole aisles dedicated to what we drink!
  
目前已经有那么多的NoSQL存储系统，多的如同运动饮料的品牌。像Facebook，Twitter，Digg，Amazon，LinkedIn和Google等知名公司也都在开发和使用NoSQL系统来解决海量数据存储问题。
  
As an IT system administrator or manager, it’s sometimes very hard to compare various NoSQL tools. It involves considering your special computing needs, matching them to what is out there, aligning what’s right for your organization and then make the right decision!
  
作为一个IT系统管理员有时很难比较各种NoSQL系统，它涉及企业的具体情景和计算需求，然后作出正确的决定！
  
That’s why Monitis, the first hosted all-in-one network and systems performance monitoring service for sysadmins, is publishing a series of blogs that are meant to offer a comprehensive guide to NoSQL technology and brands. We want to help you make the right choice that fits the particular needs of your company.
  
这就是为什么Monitis，第一个托管系统管理员所有功能于一身的网络和系统性能监控服务商，出版一系列博客，提供一个全面的NoSQL技术指南，希望能帮助您做出正确的选择。
Why should we care, you may ask yourself? Increasingly, our clients, who depend on our ability to monitorservers and networks and a host of other key metrics 24/7 from the cloud, want our advice, too, on what kind of scalable and robust database technology to use. So, we’re obliging!
  
我们为什么要关心这些，因为客户越来越多，希望我们就应用什么类型的可扩展性、稳健的数据库技术方面提供专业的意见。
  
Here, in a series of blogs, we’ll present research on existing popular NoSQL data storage tools that are generally intended to store unprecedented large amounts of data, offer flexible and horizontal scalability and provide blazing-fast processing queries. We’ll also get down to the nitty-gritty and compare several well-known NoSQL DBs…such as Cassandra, MongoDB, CouchDB, Redis, Riak, HBase and others.
  
接下来我们将对现有流行的NoSQL数据存储系统的研究进行介绍，同时我们也将分析比较几个知名的NoSQL系统... Cassandra，MongoDB，CouchDB的，Redis，Riak，HBase等。
  
In this first post, let’s discuss the reason why NoSQL technology is important.
  
在第一篇文章，我们主要讨论NoSQL技术的重要性。
  
So, just what is NoSQL anyway?
  
到底什么是NoSQL？
  
Generally, NoSQL isn’t relational, and it is designed for distributed data stores for very large scale data needs (e.g. Facebook or Twitter accumulate Terabits of data every day for millions of its users), there is no fixed schema and no joins. Meanwhile, relational database management systems (RDBMS) “scale up” by getting faster and faster hardware and adding memory. NoSQL, on the other hand, can take advantage of “scaling out” – which means spreading the load over many commodity systems.
  
一般来说，NoSQL不是关系性数据库，它是设计满足超大规模数据存储需求的分布式存储系统，没有固定的Schema，不支持join操作。传统的关系数据库管理系统通过 “向上扩展“的方式通过更好的硬件设施来提升系统性能。 NoSQL则通过“向外扩展“的方式，这意味着应用更多的普通硬件提高系统负载能力。
  
The acronym NoSQL was coined in 1998, and while many think NoSQL is a derogatory term created to poke fun at SQL, in reality it means “Not Only SQL” rather than “No SQL at all.” The idea is that both technologies (NoSQL and RDBMSs) can co-exist and each has its place. Companies like Facebook, Twitter, Digg, Amazon, LinkedIn and Google all use NoSQL in some way — so the term has been in the current news often over the past few years.
  
NoSQL的是在1998年创造的，很多人认为是SQL的反义词，其实它的真实含义是 “Not Only SQL的“而不是“No SQL”。我们认为这两种技术是可以并存的，各有自己的位置。
What’s Wrong with RDBMSs?
   RDBMS出了什么问题？  
Well, nothing, really. They just have their limitations. Consider these three problems with RDBMSs:

   其实没什么，真的。RDBMS只是有其局限性。考虑下如下的三个问题：  
1. RDBMSs use a table-based normalization approach to data, and that’s a limited model. Certain data structures cannot be represented without tampering with the data, programs, or both.
1. RDBMS中使用基于表的数据标准化方法，这是一个很受限的模型，某些非结构化的数据无法表示。    
2. They allow versioning or activities like: Create, Read, Update and Delete. For databases, updates should never be allowed, because they destroy information. Rather, when data changes, the database should just add another record and note duly the previous value for that record.  
2. 他们提供了创建，读取，更新和删除的操作。对于数据库，其实更新不会被允许的，因为他们破坏信息。当数据发生变化，数据库应该只添加另一条记录，并标记删除该记录的前一个值。  
3. Performance falls off as RDBMSs normalize data. The reason: Normalization requires more tables, table joins, keys and indexes and thus more internal database operations for implement queries. Pretty soon, the database starts to grow into the terabytes, and that’s when things slow down.    
3. 数据规范化导致性能的急剧下降。其原因是：规范化需要更多的表，连接等操作，主键、索引，从而需要更多的内部操作。很快数据库开始成长为TB级，于是性能慢下来了。   

###The Four Categories of NoSQL:
四种NoSQL类型：

Key-values Stores. The main idea here is using a hash table where there is a unique key and a pointer to a particular item of data. The Key/value model is the simplest and easiest to implement. But it is inefficient when you are only interested in querying or updating part of a value, among other disadvantages.  

1. KV存储的主要思想是一个哈希表，每个item有一个主键和特定的数据值。键/值模型是最简单，最容易实现的。但是，你在查询或更新部分值时是很低效。  
Column Family Stores.These were created to store and process very large amounts of data distributed over many machines. There are still keys but they point to multiple columns. The columns are arranged by column family.  
2.列存储主要是用来储存并处理海量数据的。仍然有主键，但它的值可以指向多个按照column Family组织的列。  
Document Databases. These were inspired by Lotus Notes and are similar to key-value stores. The model is basically versioned documents that are collections of other key-value collections. The semi-structured documents are stored in formats like JSON. Document databases are essentially the next level of Key/value, allowing nested values associated with each key.  Document databases support querying more efficiently.  
3.文档存储的灵感来自Lotus Notes和类似的键值存储，该模型的文档基本上是其他键/值的集合。半结构化文档存储在这样的JSON格式。文档数据库基本上是键/值变体，每个键可以嵌套其他值。  
Graph Databases. Instead of tables of rows and columns and the rigid structure of SQL, a flexible graph model is used which, again, can scale across multiple machines. NoSQL databases do not provide a high-level declarative query language like SQL to avoid overtime in processing. Rather, querying these databases is data-model specific. Many of the NoSQL platforms allow for RESTful interfaces to the data, while other offer query APIs.  
4.不同于行列结构化的SQL，灵活的图模型也可以扩展到多台机器。NoSQL的数据库没有像SQL提供一个高层次的声明性查询语言。许多的NoSQL的平台允许对REST风格的数据接口，而其他提供查询API。        
Generally, the best places to use NoSQL technology is where the data model is simple; where flexibility is more important than strict control over defined data structures; where high performance is a must; strict data consistency is not required; and where it is easy to map complex values to known keys.  
一般来说，最好的使用NoSQL技术的场所其数据模型简单，而且灵活性比严格的控制更重要，其中高性能是必须的，严格的数据一致性则不是必需的，并在很容易的将复杂的值映射到已知的键。  

###Some Examples of When to Use NoSQL:  
应用NoSQL的场景例子：  
Logging/Archiving. Log-mining tools are handy because they can access logs across servers, relate them and analyze them.  
1.日志/存档 日志挖掘工具是很方便，因为他们可以跨越服务器访问日志，进行关联和分析。  
Social Computing Insight. Many enterprises today have provided their users with the ability to do social computing through message forums, blogs etc.  
2.社会计算 目前许多企业透过邮件论坛，博客等为用户提供了社会计算能力。    
External Data Feed Integration. Many companies need to integrate data coming from business partners. Even if the two parties conduct numerous discussions and negotiations, enterprises have little control over the format of the data coming to them. Also, there are many situations where those formats change very frequently – based on the changes in the business needs of partners.  
  
3.外部数据集成 许多公司需要整合业务合作伙伴的数据。即使双方进行多次讨论和谈判，他们几乎还是无法控制企业拥有的数据格式。此外，还有很多情况下这些格式的变化非常频繁- 因为变动是业务合作伙伴的需求。    

Front-end order processing systems. Today, the volume of orders, applications and service requests flowing through different channels to retailers, bankers and Insurance providers, entertainment service providers, logistic providers, etc. is enormous. These requests need to be captured without any interruption whenever an end user makes a transaction from anywhere in the world. After, a reconciliation system typically updates them to back-end systems as well as updates the end user on his/her order status.
  
4.订单处理系统 服务请求数量通过不同的渠道流向零售商，银行和保险机构，娱乐服务提供商，物流供应商是巨大的。这些请求要在世界任何地方被抓获，同时保证终端用户的交易过程不受到任何影响和中断。
  
Enterprise Content Management Service. Content Management is now used across companies’ different functional groups, for instance, HR or Sales. The challenge is bringing together different groups using different meta data structures in a common content management service.

5.企业内容管理服务 内容管理被用在公司不同职能组织之间，比如人力资源或销售。目前的挑战是通过共用的内容管理服务集成不同元数据结构的不同组织。

Real-time stats/analytics. Sometimes it is necessary to use the database as a way to track real-time performance metrics for websites (page views, unique visits, etc.)  Tools like Google Analytics are great but not real-time — sometimes it is useful to build a secondary system that provides basic real-time stats. Other alternatives, such as 24/7 monitoring of web traffic, are a good way to go, too.

6.实时统计分析系统 有时需要用一种工具来跟踪网站的实时性能指标（网页浏览量，独立访问数等），如谷歌分析，可惜它不是实时的。有时建立辅助系统提供基本的实时统计信息是非常有用。其他替代办法，如24 / 7的网络流量监控，也是一个很好的办法去。

###What Type of Storage Should you use?
如何选择使用哪种存储？

Here’s a short summary that might help you make your decision:
下面是一个简短的摘要，可以帮你做决定：
NoSQL
·          Storage should be able to deal with very  high load
·          You do many write operations on the storage
·          You want storage that is horizontally  scalable
·          Simplicity is good, as in a very simple  query language (without joins)

RDBMS
·          Storage is expected to be high-load, too,  but it mainly consists of read operations
·          You want performance over a more  sophisticated data structure
·          You need powerful SQL query language

In our next series of posts, Monitis will walk you through seven popular NoSQL database tools and discuss their merits and – from our point of view – drawbacks. Stay tuned!  

在我们下一系列的文章，Monitis将深入七个流行NoSQL数据库系统，讨论其优点和缺点。请继续关注！
