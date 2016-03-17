##分布式系统的共识(consensus)算法比较
   
这是一篇比较分布式系统中服务器之间获得状态最终一致性也就是取得共识consensus几个流行算法，包括Paxos、Egalitarian Paxos、Hydra、Fast Paxos、Ios、VRR(Viewstamped Replication Revisited)、 Multi-Paxos、Raft等。

什么是共识consensus？当多个主机通过异步通讯方式组成网络集群时，这种异步网络默认是不可靠的，那么在这些不可靠主机之间复制状态需要采取一种机制，以保证每个主机的状态最终达成相同一致性状态，取得共识。

为什么认为异步网络默认是不可靠的？这是根据FLP原理。Impossibility of Distributed Consensus with One Faulty Process一文提出：在一个异步系统中我们不可能确切知道任何一台主机是否死机了，因为我们无法分清楚主机或网络的性能减慢与主机死机的区别，也就是说我们无法可靠地侦测到失败错误。但是，我们还必须确保安全可靠。

在实际中，我们首先必须接受系统可能不可用，然后试图减轻这种不可用，而不是完全回避去除，减轻的手段有定时器和补偿。Paxos等算法因此而诞生。

Paxos算法是最初最简单的分布式共识算法，它是通过节点之间来回两次实现状态复制（两段复制），但是Paxos这种来回两次的协调过程（甚至更长）拖延了时间，增加了系统的延迟。

在实际过程中，Paxos这种理想化的协议实际遭遇很多挑战，也就是说，实际中有很多问题呼唤一种分布式协调机制来处理下面这些问题：

* 处理磁盘故障和损坏
* 处理有限的存储容量
* 有效处理只读请求
* 动态成员资格和重新配置
* 支持事务
* 验证实施的安全

这就需要一种领导人或协调人角色来具体处理这些问题，Paxos是一种无领导人Leaderless算法，而Raft算法是一种强领导力Leadership的算法。总之，之所以需要协调领导人是为了确保主机之间状态复制过程的可靠性。

一个强势领导人Leadership能够在问题发生时在主机之间进行复杂的强协调，显然Leaderless则是无领导人或协调人介入处理。但是强领导在保证可靠性的同时也会聚集单点风险，而且选举领导人的过程也是费时费力的，因此，并不是领导力越强越好。需要根据自己的业务领域特点在Leadership和Leaderless之间选择合适自己的算法。

从Leadership到Leaderless以此从强到弱的排序是：

- Strong Leadership强领导：Raft

- Leader driven领导人驱动：VRR和 Mutil-Paxos

- Leader with Delegation有代表团的领导人：Ios
前面三个重视单个领导人的算法MultiPaxos,Raft 和 VRR的问题是：吞吐量将受到单个节点的限制。而Ios允许一个领导人动态地安全地委托其职责给系统中其他节点（代表团）。

- Leader only when needed只有在需要时才有领导人：Hydra和Fast Paxos

- Leaderless无领导人：Egalitarian Paxos和普通的Paxos

上述算法中，最小延迟的算法是VRR，最易于理解的是Raft，而对于广域网地区之间复制，冲突最少conflicts rare的是 Egalitarian Paxos; conflicts common是 Fast Paxos.

上述算法需要结合实际以及CAP理论与FLP综合判断选择。



当前，日益广泛使用的微服务架构会很自然地横向扩展到分布式系统，虽然微服务本身是无态的，但是它会操作有态数据，这些有态数据要么放在数据库中，要么放在缓存中，那么缓存或数据库进行横向扩展组成分布式系统时就需要使用上面这些算法，当然一些NoSQL数据库内部已经实现了这些算法，那么了解这些算法也能对我们进行NoSQL选型有参考作用。
