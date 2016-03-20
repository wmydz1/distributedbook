#传统SQL数据库不具有可扩展性


SQL Databases Don't Scale
传统SQL数据库不具有可扩展性

A question I’m often asked about Heroku is: “How do you scale the SQL database?” There’s a lot of things I can say about using caching, sharding, and other techniques to take load off the database. But the actual answer is: we don’t. SQL databases are fundamentally non-scalable, and there is no magical pixie dust that we, or anyone, can sprinkle on them to suddenly make them scale.

  我经常向Heroku问一个问题：“你是如何进行扩展SQL数据库的？”。当然你可以说出许多技巧来降低数据库的负载，比如利用缓存，数据分片等等。事实上真正的回答却是：“我们做不到。”因为从根本上讲，传统SQL数据库并不具有可扩展能力。这个世界上也根本没有什么神奇的精灵之尘，可以让我们任何人突然使其获得扩展性的。
  
What Is Scaling?
什么是可扩展性？

To qualify as true scaling, I believe a technique must fit the following criteria:

必须满足以下几个标准的才是真正合格的可扩展性，我认为这都是必要条件：

>
1. Horizontal scale: more servers creates more capacity.
   1. 水平扩展能力：更多的服务器可以提供成比例更多的容量或者更高的性能。
   2. Transparent to the application: the business logic of the app should be separated from concerns of scaling server resources.
    2. 对应用程序透明：应用程序中蕴含的业务逻辑要和关注机器资源的扩展性相分离，做到底层对上层业务的隔离，提供透明化的扩展能力。
   3. No single point of failure: there should be no one server which, if lost, causes downtime of the application.
    3. 没有单点故障问题：系统中不应该存在单点，就是说不能因为一个服务器的宕机或者故障而导致整个应用程序的不可用。
As an example from the hardware world, a RAID5 disk array offers true scaling:

____
>在硬件世界里，RAID5磁盘阵列提供了一个很好的可扩展性的例子：
1. Horizontal scale: you can run a RAID5 with 4 disks, or 12, or 20; more disks gives you more drive space and (generally) better performance.  
    1. 水平扩展：我们可以在4块，12块，甚至20块盘上应用RAID5，磁盘越多可用的空间越大，性能也越好；  
   2. Transparent to the application: applications using the RAID as a single device. My text editor doesn’t care that the file it is saving or loading is split across many disks.  
    2. 对应用程序透明：应用程序会将RAID作为一个整体，我的文本编辑器才不会关心它加载或者保存的文件是要读写多个磁盘的。  
   3. No single point of failure: you can pop out a drive and the array will continue to function (albeit with a performance hit, known as “degraded mode”). Replace the drive and it will rebuild itself. All of this happens without the applications using the RAID being aware any interruption of the functioning of the disk storage.  
    3. 没有单点故障：你可以将其中的某块盘拔出而不影响磁盘阵列继续正常服务（虽然性能会有所下降，即所谓“降级模式”），当你替换某块盘后RAID也会进行重建。对应用RAID的应用程序而言，都不会被这些情况所中断。  
So let’s take a look at some of the techniques used to “scale” SQL databases, and why they all fail to achieve the criteria above.  
    
  接下来让我们认真分析一下SQL数据库常用的一些所谓的“扩展”技巧，我们会发现这些方法并不符合前面提到的标准。
   
###Vertical Scaling
垂直扩展  
One way to scale a SQL database is to buy a bigger box. This is usually called “vertical scaling.” I sometimes call it: Moore’s law scaling. What are the problems with this?

  通常被称为“垂直扩展”的就是买一个更大的机器。我有时称其为“摩尔定律扩展Or 向上扩展”。这种方式有什么问题呢？
    
    * It’s a complicated transaction that usually requires manual labor from ops people and substantial downtime.
    1. 这是一个复杂的过程，通常需要操作人员的体力劳动和大量停机时间。
    * The old machine is useless afterward. This wastes resources and encourages you to overprovision, buying a big server before you know you’re really going to need it.
    2. 被替换的老服务器成为了废品。这种做法既浪费了资源，同时又鼓励你提前超额预支。我们提倡的是：在买一个大的服务器之前，你知道你真的需要它。

    * There’s a cap on how big you can go.
    3. 这样的扩展是及其有限的。
  
I know plenty of folks who have bumped their head on the last point (usually somewhere around a 256-core Sun server) and now they are painted into a corner. They find themselves crossing their fingers and hoping that they’ll stop growing - not cool at all.

  我就认识很多人都在最后一点上栽了跟头（通常也就是256个核的SUN服务器），现在都痛苦不堪，只能双手合十，寄希望于负载不再增长，一点都不酷。
  
So yes, you can put a SQL database on a bigger box to buy yourself more headroom. But this fails point #1 on my checklist for true scaling.
    是的，你还是可以将SQL数据库搭在更强的服务器上，从而提供更大的空间，但是这的确违背了我提到的关于真正的可扩展性的第一条标准。


###Partitioning, aka Sharding
数据分片  
Sharding divides your data along some kind of application-specific boundary. For example, you might store users whose names start with A-M on one database, and N-Z on another. Or use a modulo of the user id by the number of databases.  

  这种方式一般是根据系统某种特定的边界限制对数据进行划分，比如将姓名首字母为A-M的用户分到一个库，N-Z划分到另一个库，或者根据用户ID对数据库的数目进行取模运算。
This requires deep integration into the application and careful planning of the partitioning scheme relative to the database schema and the kinds of queries you want to do. Summary: big pain in the ass.

  它紧密依赖于应用并且你需要仔细的去计划你的schema，总之：会很蛋疼，绝不是闲的.
So while sharding is a form of horizontal scaling, it fails point #2: it is not transparent to the business logic of the application.

   因此，分片尽管是一种水平扩展方式，但是它违背了第二点，即对上层应用程序的业务逻辑没有做到透明。
 
The deeper problem with sharding is that SQL databases are relational databases, and most of the value in a relational database is that it stores relationships. Once you split records across multiple servers, you’re servering many of those relations; they now have to be reconstructed on the client side. Sharding kills most of the value of a relational database.

  此外，采用分片方式存在更深层的问题，SQL数据库是关系型，其存储的大部分都是存在关系的数据内容。一旦将这些数据记录分割到多台服务器，你需要维护这些关系，并且需要在客户端重建这些关系，因此分片也同时抹杀了关系型数据库最大的优点。
 
###Read Slaves
读写分离

MySQL’s killer feature is easy configuration of master-slave replication, where you have a read-only slave database that replicates everything coming to the master database in realtime. You can then create a routing proxy between the clients and your database (or build smart routing into the client library) which sends any reads (SELECT) to one of the read slaves, while only sending writes (INSERT, UPDATE, DELETE) to the master.  

  MySQL堪称杀手级的特性就是可以十分方便的配置主从同步复制模式，从而便利的提供只读功能而且又可以实时和主库保持更新同步的从库。你可以通过创建一个位于数据库和应用程序之间的路由代理完成读写分离，并对上层提供透明的访问接口。  
    
Postgres has replication via Slony, though it’s much more cumbersome to set up than MySQL’s replication. Years ago I even did basic master-slave replication with a 50-line Perl script that read from the query log and mirrored all write queries over to the slave. So replication is possible just about anywhere, with differing degrees of setup and maintenance headaches.
  
  Postgres 也可以通过Slony进行主从同步复制，但是安装配置相比MySQL要复杂一些。很多年前，我甚至写了一个50行左右的Perl脚本来完成主从之间的同步，这个脚本很简单，读取操作日志，并将更新日志同步到从库。因此，其可以毫无差别和维护难度的将复制应用到任何地方。
The read slave technique is the best option for scaling SQL databases. It qualifies as horizontal scaling on the read side, and is transparent to the application. This is the technique that many of the largest MySQL installs use.
   
   从库技术是最好扩展SQL数据库的选项，即横向扩展了读，而且可以做到对上层的透明化。因此，许多大型的MySQL应用都使用了主从同步复制。

And yet, this is still ultimately a limited technique. The master server is a bottleneck, particularly on write-heavy applications. And it fails point #3 on the true scaling definition: there is a single point of failure. This is a problem not only when the database fails, but when you want to perform maintenance on the server. Promoting one of the read slaves to master allows you to recover relatively quickly, but this switcharoo requires hands-on attention from the sysadmins.
   
   但是，这仍然有很多的限制。由于只有主库提供写服务，因此在写负载很重的场景里主库将成为系统的瓶颈。这违反了第三点标准：系统存在单点。不仅是在主库宕机的时候，当你需要对服务器进行维护的时候，这都成为了问题。将只读的从库提升为主库可以使尽快恢复服务，但是这种切换需要系统管理员的关注和动手。
 
 And while it qualifies as horizontal scaling for read performance, it does not qualify for writes and capacity. Horizontal scaling should spread out both the load of executing queries and the data storage itself. This is how you can grow the total storage space of a RAID: add enough disks, and you can get a RAID which has a much greater capacity than any single hard drive on the market. Read slaves require complete copies of the database, so you still have a ceiling on how much data you can store.
    
   同时，从库对读性能具备了水平扩展能力，但是对于写和存储容量而言作用不大什么没有什么扩展。水平扩展本应该同时提高读写负载和数据存储容量，就像RAID一样，加上更多的磁盘就可以拥有更大的存储空间。但是提供只读服务的从库是对数据的完全拷贝，因此仍然存在数据存储在容量上的天花板。

###A Long-Term Solution
长远的解决之道

So where do we go from here? Some might reply “keep trying to make SQL databases scale.” I disagree with that notion. When hundreds of companies and thousands of the brightest programmers and sysadmins have been trying to solve a problem for twenty years and still haven’t managed to come up with an obvious solution that everyone adopts, that says to me the problem is unsolvable. Like Kirk facing the Kobayashi Maru, we can only solve this problem by redefining the question.
  
  那我们该何去何从呢？有些人会说：“努力让SQL数据库具有可扩展性。”我对此持否定意见，成千的公司以及上万聪明的程序员和管理员通过20多年的努力，仍然没有找到一种明显的而且让大家都接受的解决方法，这意味着什么，对我而言就是这是无解的一个问题。就像Kirk面对Kobayashi Maru一样,我们唯一可以做的就是重新定义这个问题。
    