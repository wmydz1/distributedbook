#什么是系统的可扩展性？

When asked what they mean by scalability, a lot of people talk about improving performance, about implementing HA, or even talk about a particular technology or protocol. Unfortunately, scalability is none of that. Don’t get me wrong. You still need to know all about speed, performance, HA technology, application platform, network, etc. But that is not the definition of scalability.

   每每和别人提及可扩展性的含义时，很多人开始讨论提高性能，实施高可用性，甚至谈论特定的技术或协议。显然这些并不是可扩展性。不要误会，您当然需要了解关于速度，性能，可用性，应用平台，网络等相关的一切，但这并非可扩展性的定义。
   
Scalability, simply, is about doing what you do in a bigger way. Scaling a web application is all about allowing more people to use your application. If you can’t figure out how to improve performance while scaling out, its okay. And as long as you can scale to handle larger number of users its ok to have multiple single points of failures as well.

   简单地说，可扩展性就是关于如何处理更大规模的业务。比如，Web应用程序就是允许更多的人使用你的服务。如果你不能弄清楚如何提高性能的同时向外扩展，没关系。只要你能处理更大规模的用户，即使是存在多个单点故障也没有问题。
   
There are two key primary ways of scaling web applications which is in practice today.

  在今天实践中有两个关键的缩放Web应用程序的方式：垂直扩展和水平扩展。
  
“Vertical Scalability” – Adding resource within the same logical unit to increase capacity. An example of this would be to add CPUs to an existing server, or expanding storage by adding hard drive on an existing RAID/SAN storage.
   
   “垂直扩展“ -在同一个逻辑单位添加资源以增加容量。这样的例子比比皆是，比如升级服务器的CPU，比如在RAID/ SAN存储设备上增加硬盘。
“Horizontal Scalability” – Adding multiple logical units of resources and making them work as a single unit. Most clustering solutions, distributed file systems, load-balancers help you with horizontal scalability.

   “横向扩展“- 增加多个逻辑单元资源并且使他们作为一个整体在工作。大多数的集群解决方案，比如分布式文件系统，负载均衡都是通过横向扩展技术来进行的。
   
Every component, whether its processors, servers, storage drives or load-balancers have some kind of management/operational overhead. When you try to scale that, its important to understand what percentage of the resource is actually usable. This measurement is called “scalability factor“. If you loose 5% of a processor power every time you add a CPU to your system, then your “scalability factor” is 0.95. A scalability factor of 0.9 means you will only be able to use 90% of the resource.

   每一个部件，无论它是处理器，服务器，存储驱动器或负载均衡有一定的管理上或者操作上的开销。当您尝试进行扩展时，很重要的一点是要了解实际的资源利用率，这种测量方法被称为“可扩展性因子“法。如果每添加一个CPU到系统，都会失去5％的处理器功率，那么您的可扩展性系数为0.95。当可扩展系数为0.9时，意味着你将只能使用90％的资源。
   
Scalability can be further sub-classified based on the “scalability factor”.
       
   以“可扩展性因子“为基础，可扩展性可以进一步细化分类。
   
If the scalability factor stays constant as you scale. This is called “linear scalability“.

   如果你的可扩展性系数保持不变，这就是所谓的“线性的可扩展性“。
   
But chances are that some components may not scale as well as others. A scalability factor below 1.0 is calleub-linear scalability“.

   但某些组件有可能无法像别人一样可以扩展，该系数低于1.0的就是所谓“分线性可扩展性。“
   
Though rare, its possible to get better performance (scalability factor) just by adding more components (i/o across multiple disk spindles in a RAID gets better with more spindles). This is called “supra-linear scalability“.

   虽然数量很少，但是加入更多的组件确实可能会获得更好的性能（或者可伸缩性因子）（跨越多个磁盘的RAID I / O性能确实会更好）。这就是所谓的“超线性的可扩展性。“
   
If the application is not designed for scalability, its possible that things can actually get worse as it scales. This is called “negative scalability“.

   如果应用程序没有可扩展性，当对其进行扩展时，得到了更加糟糕的结果。这就是所谓的“负可扩展性“。
   
If you need scalability, urgently, going vertical is probably going to be the easiest (provided you have the bank balance to go with it). In most cases, without a line of code change, you might be able to drop in your application on a super-expensive 64 CPU server from Sun or HP and storage from EMC, Hitachi or Netapp and everything will be fine. For a while at least. Unfortunately Vertical scaling, gets more and more expensive as you grow.
   
   如果您迫切需要可扩展性，垂直扩展可能是最容易的（只要你有足够的银行存款余额）。在大多数情况下，只要能砸一个超级昂贵的64个来自Sun或HP CPU服务器和来自EMC，NetApp和日立的存储设备，不用改变一行代码，一切都OK，至少可以抗一段时间。不幸的是伴随您的成长，垂直扩展会变得越来越贵。
   
Horizontal scalability, on the other hand doesn’t require you to buy more and more expensive servers. Its meant to be scaled using commodity storage and server solutions. But Horizontal scalability isn’t cheap either. The application has to be built ground up to run on multiple servers as a single application. Two interesting problems which most application in a horizontally scalable world have to worry about are “Split brain” and “hardware failure“.
        
   另一方面，横向扩展并不要求你购买更多昂贵的服务器，这意味着应用普通的存储和服务器作为解决方案。但是，横向扩展也不便宜，因为它需要应用程序进行重新设计，以运行在多个服务器之上，但是对外要表现为一个单一的应用程序。这其中不得不思考两个有趣的问题：“脑裂“和”硬件故障“。
        
While infinite horizontal linear scalability is difficult to achieve, infinite vertical scalability is impossible. If you are building capacity for a pre-determined number of users, it might be wise to investigate vertical scalability. But if you are building a web application which could be used by millions, going vertical could be an expensive mistake.

   虽然无限的水平线性可扩展性是难以实现的，无限的垂直扩展性也是不可能的。如果你正在构建一个基本预先可以确定用户数量的系统，调查和应用垂直扩扩展可能是明智的选择。但是，如果你正在建设一个可用于数百万用户的Web应用程序，垂直扩展可能要付出昂贵的代价。
        
But scalability is not just about CPU (processing power). For a successful scalable web application, all layers have to scale in equally. Which includes the storage layer(Clustered file systems, s3,etc), the database layer (partitioning,federation), application layer(memcached,scaleout,terracota,tomcat clustering,etc), the web layer, loadbalancer , firewall, etc. For example if you don’t have a way to implement multiple load balancers to handle your future web traffic load, it doesn’t really matter how much money and effort you put into horizontal scalability of the web layer. Your traffic will be limited to only what your load balancer can push.

   可扩展性不仅是和CPU处理能力有关，对于一个成功的具备可扩展性的Web应用程序而言，所有层都是一样的，这包括存储层（集群文件系统和S3等），数据库层（分区，联邦），应用层（memcached，横向扩展，TC，Tomcat集群等），网络层，负载平衡器，防火墙等等。例如如果没有办法实现多个负载平衡器来处理您未来的网络流量负载，这和您花多少钱和精力以使你的web层具备横向扩展能力毫无关系，您的流量将限于负载平衡器的能力。
        
Choosing the right kind of scalability depends on how much you want to scale and spend. In fact if someone says there is a “one size fits all” solution, don’t believe them. And if someone starts a “scalability” discussion in the next party you attend, please do ask them what they mean by scalability first.
   
   选择合适的可扩展方法取决于你想要的规模和预算。事实上如果有人说有一个“放之四海而皆准“的解决办法，请不要相信他们。如果下次您参加聚会讨论时，有人开始“可扩展性“这个话题，请首先问他们可扩展性的含义。
        
References
参考文章

-  Cost and Scalability
- My linear scalability is bigger than yours