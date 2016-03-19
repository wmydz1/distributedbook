#基于Lease的一致性

基于Lease的一致性最初应用于分布式文件Cache，后来随着互联网的快速发展，发现非常适合于Web Proxy，因此针对Proxy Cache领域中的Lease便多了起来。

###1. Lease的由来
关于Lease最经典的解释来源于Lease的原始论文

<<Leases: An Efficient Fault-Tolerant Mechanism for Distributed File Cache Consistency>>：

####a lease is a contract that gives its holder specific rights over property for a limited period of time

即Lease是一种带期限的契约，在此期限内拥有Lease的节点有权利操作一些预设好的对象，一般把拥有Lease节点称为Master。从更深 层次上来看，Lease就是一把带有超时机制的分布式锁，如果没有Lease，分布式环境中的锁可能会因为锁拥有者的失败而导致死锁，有了lease死锁 会被控制在超时时间之内。
###2. Web Server Proxy
众所周知，一般会采用Proxy的方式加速对Web资源的访问速度，而Proxy也是HTTP协议里面的一个标准组件，其基本原理就是对远程 Server上的资源进行Cache，Client在访问Proxy时如果所需的内容在Proxy Cache中不存在，则访问Server；否则直接把Cache中的内容返回给Client。通过Proxy既提高了用户体验，又降低了Server负 载。当然，一个Server会存在很多个Proxy。 

因此，保证Cache中的数据与Server一致成为Proxy Cache能正确工作的前提。之前，一般互联网数据不需要很强的一致性，但随着支付、股票等应用的发展，强一致性成为比不可少的要求，Lease就是解决这类强一致性的折中方案。 

在不需要强一致性的环境中，Cache只要每隔一段时间与Server同步一下即可，但在需要强一致性的环境这种做法远不能满足需求。一般的实现强一致性有下面两种方式： 

- 客户端驱动：每次Read之前都check，像HTTP协议的If-Modified-Since头
- Server驱动：Server上面的每次变更都通知Cache，这种方式具体又有两种实现：
    1.1. Server仅通知Cache数据失效，Cache主动把失效数据拉过来 
    1.2. Server把变更的数据通知Cache
 
基于客户端的好处是，同步过程与Server完全无关，两套系统没有耦合，维护性强，但会造成很大的无效流量，Server要承担很大的负载。基于 Server的好处是没有无效流量，每次更新都很准确，但通信量太大，不需要的更新的内容也被更新。Server驱动还有另外一个问题，就是如果某个 Proxy无法响应，Server可能会陷于死锁状态，从而影响其他Cache的正常工作。

###3.应用Lease到Proxy
引入Lease后规定：

- Cache第一次访问Server时，Server返回该Request的内容与一个Lease（Cache称为Holder）
- 在Lease期限之内，Server会主动把更新数据推送到Cache（是推送失效通知还是变更数据要看应用的情况）

- 如果Lease超时，Cache需要重新申请Lease

如果Lease时间无穷大，就是Server驱动模式；如果为0，则为客户端驱动模式，因此Lease是二者的一个折中。 

很明显，影响Lease效果的一个重要的一个参数是Lease的时间：太长，Server端需要为Request予准备很多的状态（占用很多的空 间）；太小，则会造成Cache与Server之间的流量增大，加大Server负载。在专业论文中用很多的公式来描述这个参数的影响，感觉没多大必要， 这里略过不表。

Lease模式可以称为“按需一致”，即用户访问的数据才进行一致，而对那些没有访问的数据不需要一致，这就一定程度上克服了Server驱动模式 下全部推送数据的缺点。试想有多个Proxy的场景，只要某条数据在多个Proxy上访问过，那么他们在所有的Proxy上是一致的，否则可能会不一致。

###4. Lease的特点
很显然，Lease更擅长解决Cache与某个数据源之间的数据一致性问题，而不关心多Cache之间数据是否一致，其应用场景有较强的限制，非常 适合与类似Proxy这样的情况。但在真正Proxy Cache实现中中会根据不同的数据，同时存在弱一致性和Lease强一致性，也称为可调节的一致性。 

Lease算法看似简单，但为我们在其他处理分布式问题时提供了很好的思路。尤其是原论文作者对Lease的定义，其价值远大于其在Proxy中的应用。
###5.Lease演化
很多人沿着Lease这条路继续向下走，又产生了： 

- volume lease：把对象作为一个集合
- hierarchical lease：把Lease层次化

这些算法的目标都是在特定场景下寻找平衡lease时间的方法。
因为我们着重研究Lease的一致性，对其在Web环境下的具体应用讨论的并不多，感兴趣的同学可以参考：

Adaptive Leases: A Strong ConsistencyMechanism for the World Wide Web

Lease Based Consistency Scheme in the Web Environment.pdf
