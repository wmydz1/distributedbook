#一致性、可用性和收敛性(CAC)

　　一致性Consistency, 可用性Availability, 和收敛性Convergence是分布式系统中相对于CAP定理的另外一个定理，2014年由Mahajan, Alvisi, 和 Dahlin提出： [Consistency, Availability, and Convergence](http://www.cs.utexas.edu/users/dahlin/papers/cac-tr.pdf) 。

　　CAP(consistency, availability, partition)混合了分布式特性(如一致性和可用性)与系统模型(网络可靠性指标)，在CAC中，则将这些分布式特性与系统模型进行了分离。

 

收敛性

　　在经典的最终一致性模型中有一些无用的模型，比如所有分布式节点都会一致返回一个常量值，Mahajian他们通过引入收敛性这个符合我们常识定义来修正了这些漏洞。

　　CAP为什么没有明确考虑收敛性？是因为线性化和顺序这两种一致性里面已经包含了收敛性的需求，当我们检查如因果一致性 causal consistency，我们会发现我们必须明确地考虑收敛性。

　　收敛性是指一种实现能力，它能确保被一个节点写入的数据被另外一个读取，收敛性的定义是：描述的是一个节点能够读取到其他节点的写入时的一系列环境条件(如网络，本地时钟等)。

　　一个简单的收敛性其实是一种最终一致性，如果一个系统停止了接受写入和足够的通讯发生，那么这个系统就会达到一种状态，这种状态是，对于任何对象o，o的读取会在所有节点上返回同样的值。

　　在节点A和B之间的单边收敛 one way convergence是指：使用两步单向通讯完成收敛性，首先 A将修改发往B，然后B将修改发往A。

　　如果说，一致性是指所有节点都同意，那么收敛性是指所有节点都同意是一种可取的有用的状态。

　　通过引入收敛性，我们可以在安全（一致性）和灵活性（可用与收敛）之间取得平衡。

 

因果一致性

　　因果一致性(Causal consistency)遵循‘happens-before’ 图义，也就是说，写在读之前发生，实时因果一致性(RTC)是增加了时间不可逆的约束。

　　没有一致性比实时因果一致性(RTC)更强了，RTC能一边提供可用性，一边提供收敛性系统。

　　RTC的实现类似日志交换log-exchange协议，每个写操作会产生一个带有向量时钟vector clock 或版本向量、对象标识和对象值三者结合的更新，向量时钟决定更新的优先权，每个节点上的本地存储跟踪对每个对象的最近更新，当读取一个对象o时，节点不需要任何通讯情况下从本地存储返回最近更新给o，类似地，写操作更新会被创建和加入到本地存储和本地日志，节点之间会定期从它们的本地日志中交换这种更新。最新接受的更新会被追加到本地日志，这样能够用于更新节点的本地存储，替换任何旧的更新，将在因果上优先于新的更新。

　　在我们实现中，每个节点定期发送它的日志到所有其他节点以确保单边收敛， 这种实现不需要任何节点之间通讯能确保读写完成，它是单边收敛的原因是，因为接受来自一个发送者的更新能应用到接受者获得收敛状态，节点之间会定期广播它们本地日志的所有更新。同时这也是因果一致性，因为向量时钟携带了每次更新，能确保最新的写操作被读操作获得（时间上先后）。最后，它是RTC实现，因为向量时钟分配不会违反实时性要求，比如通过分配一个旧的向量时钟给一个较新的更新。
