#8种Nosql数据库系统对比

尽管SQL数据库是非常有用的工具，但经历了15年的一支独秀之后垄断即将被打破。这只是时间问题：被迫使用关系数据库，但最终发现不能适应需求的情况不胜枚举。  
  
但是NoSQL数据库之间的不同，远超过两个 SQL数据库之间的差别。这意味着软件架构师更应该在项目开始时就选择好一个适合的 NoSQL数据库。

针对这种情况，这里对 Cassandra、 Mongodb、CouchDB、Redis、 Riak、Membase、Neo4j和HBase进行了比较：
  
1.      CouchDB（V1.1.0）

l  语言： Erlang  
l  特点：DB一致性，易于使用  
l  使用许可： Apache  
l  协议： HTTP/REST  
l  双向数据复制， 
l  持续性或ad-hoc  
l  冲突检测  
l  采用master-master复制  
l  MVCC – 写操作不阻塞读操作  
l  版本控制（之前的版本文档有效）  
l  单点崩溃（可靠的）设计  
l  必要时可进行数据压缩  
l  视图：内嵌 map/reduce 机制(MapReduce是一种编程模型，用于大规模数据集的并行运算-译注)  
l  视图格式：列表&显示  
l  支持进行服务器端文档验证  
l  支持身份验证  
l  数据变化实时更新  
l  支持附件处理  
l  CouchApps（独立的 js应用程序）  
l  包含jQuery程序库  

最佳应用场景：适用于累积大量数据，变化较少，执行预定义查询的应用。适用于需要提供数据版本支持的应用。
例如： CRM、CMS系统。Master-master复制是一个有趣的功能，很易于进行多站点部署。
  
-----------------------------------------------------------------------
    
2.      Redis（V2.4） 
 
l  语言：C/C++  
l  特点：速度极快  
l  使用许可： BSD  
l  协议：Telnet-like  
l  有硬盘存储支持的内存数据库，  
l  目前还没有磁盘交换（VM和Diskstore被抛弃）  
l  Master-slave复制   
l  通过key进行简单的值存储或哈希表存储  
l  支持复杂操作，例如 ZREVRANGEBYSCORE。  
l  INCR & co （数字递增存储键值，适合计算极限值或统计数据）  
l  支持集合运算（支持 交集/差集/子集）  
l  支持列表（支持队列；阻塞式 pop操作）  
l  支持哈希表（多个域的对象）  
l  支持集合排序（高分表，适用于范围查询）  
l  支持事务  
l  支持将数据设置成过期数据（类似快速缓冲区设计，以及冷热分离）  
l  发布/订阅功能允许用户实现消息机制（长连接推送机制）  
 最佳应用场景：适用于数据变化快且数据库大小可以预计的应用程序。（这样可以合理配置内存容量）  
例如：股票价格、数据分析、实时数据搜集、实时通讯。
  
-----------------------------------------------------------------------
    
3.   MongoDB
l  语言：C++  
l  特点：保留了SQL一些友好的特性（查询，索引）。  
l  使用许可： AGPL（发起者： Apache）  
l  协议： Custom, binary（BSON）  
l  Master/slave复制（服务器间数据复制和自动故障转移）  
l  内建自动分片机制（支持水平数据库集群）  
l  支持 javascript表达式查询  
l  可在服务器端执行任意的 javascript函数  
l  优于CouchDB的update-in-place  
l  采用内存映射文件的方式进行数据存储  
l  对性能的要求高于功能  
l  最好打开日志功能（参数 –journal）  
l  在32位操作系统上，数据库大小限制约为2.5G  
l  空数据库大约占 192Mb  
l  采用 GridFS存储大数据和元数据（不是真正的文件系统）

 最佳应用场景：如果你需要动态查询，需要使用索引而不是 map /reduce功能；需要对大数据库有良好的性能要求。如果你想使用 CouchDB，但数据改变太频繁而占满内存的应用程序。
例如：你本打算采用 MySQL或 PostgreSQL，但因为需要预定义表结结构让你望而却步。
  
-----------------------------------------------------------------------
    
4. Riak（V1.0）
l  语言：Erlang和C，以及一些Javascript  
l  特点：具备容错能力  
l  使用许可： Apache  
l  协议： HTTP/REST或者 custom binary  
l  可自定义参数控制的分布和复制(N-复制节点数, R- 成功读操作的最小节点数, W–成功写操作的最小节点数-译注)  
l  用 JavaScript或 Erlang在操作预提交或提交时进行验证和安全检测  
l  使用JavaScript或Erlang进行 Map/reduce  
l  链接&链接遍历：可作为图形数据库使用  
l  多级索引：可在元数据中进行搜索
l  大数据对象支持（Luwak）  
l  提供“开源”和“企业”两个版本  
l  Riak搜索服务器（beta版）支持全文搜索、索引、查询  
l  正在将后端存储从“Bitcask”迁移到Google的“LevelDB”  
l  支持Masterless多站点复制及商务授权的 SNMP监控  
 最佳应用场景：适用于想使用类似 Cassandra（类似Dynamo，亚马逊的key-value模式的存储平台-译注）数据库但又不愿处理数据臃肿及复杂性的情况。如果你需要很好的单站点可伸缩性，可用性和容错性，但又准备实行多点复制。  
例如：销售数据搜集，工厂控制系统；对宕机时间有严格要求；可以作为易于更新的 web服务器使用。  
  
-----------------------------------------------------------------------
    
5.  Membase  
l  语言： Erlang和C  
l  特点：兼容 Memcache，兼具持久化和支持集群  
l  使用许可： Apache 2.0  
l  协议：Memcached 扩展增强  
l  非常快速（200k+/秒），通过键值访问数据  
l  可持久化存储到硬盘  
l  所有节点都是相同的（Master-master复制）  
l  在内存中提供类似Memcached 的缓存单元  
l  通过重复数据删除技术写入数据来减少IO  
l  提供良好的集群管理web界面  
l  软件更新时无需停止数据库服务  
l  支持连接池和多路复用的连接代理    

 最佳应用场景：适用于需要低延迟数据访问，高并发和高可用性的应用  
例如：低延迟数据访问比如以广告为目标的应用，高并发的web 应用比如网络游戏（如Zynga）

-----------------------------------------------------------------------
  
6.  Neo4j （V1.5M02）  
l  语言： Java  
l  特点：图形数据库  
l  使用许可： GPL，其中一些特性使用AGPL/商业许可  
l  协议： HTTP/REST（或嵌入在 Java中）  
l  可独立使用或嵌入到 Java应用程序  
l  完全符合ACID特性（包括持久化数据）  
l  图形的节点和边都可以带有元数据  
l  集成的基于模式匹配的查询语言（Cypher）  
l  可以使用图形遍历语言”Gremlin”  
l  节点和关系的索引  
l  内建练好的web管理界面  
l  多算法支持的高级路径查找  
l  key和关系索引  
l  优化的读取操作  
l  支持事务（Java api）  
l  支持 Groovy脚本  
l  支持在线备份，高级监控及高可靠性支持，使用AGPL/商业许可  
 
最佳应用场景：适用于描述图形类数据、丰富数据或者复杂数据之间的关系。这是 Neo4j与其他nosql数据库的最显著区别  
例如：社会关系，公共交通网络，地图，网络拓扑等
    
-----------------------------------------------------------------------
    
7.  Cassandra  
l  语言： Java  
l  特点：对BigTable和 Dynamo(亚马逊的key-value模式的存储平台-译注)支持得最好  
l  使用许可： Apache  
l  协议： Custom, binary (Thrift)  
l  可自定义参数控制的分布和复制(N-复制节点数, R- 成功读操作的最小节点数, W–成功写操作的最小节点数)  
l  支持以某个范围的键值查询，列查询  
l  类似BigTable的功能：列，列组  
l  写操作比读操作更快  
l  基于 Apache Hadoop(一个分布式系统基础架构，由Apache基金会开发-译注)尽可能地进行Map/reduce  
我承认对 Cassandra有偏见，因为它本身的臃肿和复杂性，当然部分是因为 Java的问题（配置，出现异常，等等）   
最佳应用场景：当写操作多于读操作（记录日志）时。如果系统中的每个组件都必须用Java编写（没有人因为选用 Apache的软件被解雇）。  
例如：银行业，金融业（虽然对于金融交易不是必须的，但这些产业对数据库的要求会比它们更大）写比读更快，所以一个自然的特性就是实时数据分析  
  
-----------------------------------------------------------------------
    
8.  HBase
（配合 ghshephard使用）  
l  语言： Java  
l  特点：支持数十亿行 × 上百万列  
l  使用许可： Apache  
l  协议：HTTP/REST （支持 Thrift，见编注4）
l  以BigTable为蓝本  
l  使用Hadoop进行Map/reduce  
l  通过在server端扫描及过滤实现对查询操作预判  
l  实时查询优化  
l  高性能 Thrift网关  
l  支持 XML, Protobuf, 和binary的HTTP  
l  Cascading, hive, 以及 pig source 和 Sink 模块  
l  基于Jruby（JIRB）的shell  
l  不会出现单点故障  
l  对配置改变和较小的升级都会重新回滚  
l  堪比MySQL的随机访问性能  
 最佳应用场景：适用于偏好BigTable，并且需要对大数据进行随机、实时访问的应用。
例如： Facebook消息数据库（更多通用用例即将出现）  
  
----------------------------------------------------------------------
当然，所有的系统都不只具有上面列出的这些特性。这里我仅仅根据自己的观点列出一些我认为重要的特性。与此同时，技术进步是飞速的，所以上述的内容肯定需要不断更新。我会尽我所能地更新这个列表。
 
原文：Kristóf Kovács 　