CAP理论以及Eventually Consistent （最终一致性）解析

- CAP理论简介

10年前，Eric Brewer教授指出了著名的CAP理论，后来Seth Gilbert 和 Nancy lynch两人证明了CAP理论的正确性。CAP（Consistency,Availability,partition tolerance)理论告诉我们，一个分布式系统不可能满足一致性，可用性和分区容错性这三个需求，最多只能同时满足两个（至于CAP理论的证明，可以参考附件的论文）。

在CAP理论的指导下，架构师或者开发者应该清楚，您当前架构和设计的系统真正的需求是什么？您的系统到底是关注的是可用性还是一致性的需求。如果您的系统关注的是一致性，那么您就需要处理因为系统不可用而导致的写操作失败的情况，而如果您关注的是可用性，那么您应该知道系统的read操作可能不能精确的读取到write操作写入的最新值。因此系统的关注点不同，相应的采用的策略也是不一样的，只有真正的理解了系统的需求，才有可能利用好CAP理论。

- 一致性 

说起一致性，我们有两种视角去看待它，第一种是客户或者开发者视角，此种情况下，客户或者开发者更加关注：如何观察到系统的更新，另外一种视觉是服务器端视觉，此种情况下，主要关注：更新操作如何flow through系统以及系统对更新操作提供什么样子的一致性保证。

下面就分别从这两种角度去看看一致性：

### 客户端一致性

为了更好的描述客户端一致性，我们通过以下的场景来进行，这个场景中包括三个组成部分：

存储系统

存储系统可以理解为一个黑盒子，它为我们提供了可用性和持久性的保证。

Process A

ProcessA主要实现从存储系统write和read操作

Process B 和ProcessC 

ProcessB和C是独立于A，并且B和C也相互独立的，它们同时也实现对存储系统的write和read操作。

下面以上面的场景来描述下不同程度的一致性：

强一致性（即时一致性）

假如A先写入了一个值到存储系统，存储系统保证后续A,B,C的读取操作都将返回最新值

弱一致性

假如A先写入了一个值到存储系统，存储系统不能保证后续A,B,C的读取操作能读取到最新值。此种情况下有一个“不一致性窗口”的概念，它特指从A写入值，到后续操作A,B,C读取到最新值这一段时间。

最终一致性

最终一致性是弱一致性的一种特例。假如A首先write了一个值到存储系统，存储系统保证如果在A,B,C后续读取之前没有其它写操作更新同样的值的话，最终所有的读取操作都会读取到最A写入的最新值。此种情况下，如果没有失败发生的话，“不一致性窗口”的大小依赖于以下的几个因素：交互延迟，系统的负载，以及复制技术中replica的个数（这个可以理解为master/salve模式中，salve的个数），最终一致性方面最出名的系统可以说是DNS系统，当更新一个域名的IP以后，根据配置策略以及缓存控制策略的不同，最终所有的客户都会看到最新的值。

其中最终一致性（Eventually consistency)还有以下几个变体，下面分别描述：

Causal consistency（因果一致性）：如果Process A通知Process B它已经更新了数据，那么Process B的后续读取操作则读取到A写入的最新值，而与A没有因果关系的C则采用最终一致性的语义。

Read-your-writes consistency：如果Process A写入了最新的值，那么Process A的后续操作都会读取到最新值。比如以本论坛为例，当某用户发表一个帖子以后，该用户可以立即看到自己发表的帖子，但是其它用户可能要过一会才可以看到，这就是典型的Read-your-writer-consistency。


Session consistency:此种一致性要求客户端和存储系统交互的整个会话阶段保证Read-your-writes consistency.Hibernate的session提供的一致性保证就属于此种一致性。

Monotonic read consistency：此种一致性要求如果Process A已经读取了对象的某个值，那么后续操作将不会读取到更早的值。

Monotonic write consistency：此种一致性保证系统会序列化执行一个Process中的所有写操作。


### 服务器端一致性

在说服务器端一致性要求，我们首先要明确几个概念：
N： 节点的总个数
W 更新的时候需要确认已经被更新的节点的个数
R： 读取数据的时候读取数据的节点个数

如果W+R>N，那么分布式系统就会提供强一致性的保证，因为读取数据的节点和被同步写入的节点是有重叠的。在一个RDBMS的复制模型中（Master/salve)，假如N=2,那么W=2,R=1此时是一种强一致性,但是这样造成的问题就是可用性的减低，因为要想写操作成功，必须要等2个节点都完成以后才可以。

在分布式系统中，一般都要有容错性，因此一般N都是大于3的，此时根据CAP理论，一致性，可用性和分区容错性最多只能满足两个，那么我们就需要在一致性和分区容错性之间做一平衡，如果要高的一致性，那么就配置N=W，R=1,这个时候可用性就会大大降低。如果想要高的可用性，那么此时就需要放松一致性的要求，此时可以配置W=1，这样使得写操作延迟最低，同时通过异步的机制更新剩余的N-W个节点。

当存储系统保证最终一致性时，存储系统的配置一般是W+R<=N,此时读取和写入操作是不重叠的，不一致性的窗口就依赖于存储系统的异步实现方式，不一致性的窗口大小也就等于从更新开始到所有的节点都异步更新完成之间的时间。

无论是Read-your-writes-consistency,Session consistency,Monotonic read consistency,它们都通过黏贴（stickiness)客户端到执行分布式请求的服务器端来实现的，这种方式简单是简单，但是它使得负载均衡以及分区容错变的更加难于管理，有时候也可以通过客户端来实现Read-your-writes-consistency和Monotonic read consistency,此时需要对写的操作的数据加版本号，这样客户端就可以遗弃版本号小于最近看到的版本号的数据。

在系统开发过程中，根据CAP理论，可用性和一致性在一个大型分区容错的系统中只能满足一个，因此为了高可用性，我们必须放低一致性的要求，但是不同的系统保证的一致性还是有差别的，这就要求开发者要清楚自己用的系统提供什么样子的最终一致性的保证，一个非常流行的例子就是web应用系统，在大多数的web应用系统中都有“用户可感知一致性”的概念，这也就是说最终一致性中的“一致性窗口"大小要小于用户下一次的请求，在下次读取操作来之前，数据可以在存储的各个节点之间复制。还比如假如存储系统提供了read-your-write-consistency一致性，那么当一个用户写操作完成以后可以立马看到自己的更新，但是其它的用户要过一会才可以看到更新。

以上是这篇文章核心的总结：
[Eventually Consistency](http://www.allthingsdistributed.com/2008/12/eventually_consistent.html)