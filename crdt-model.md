#分布式CRDT模型

　　在分布式系统中，我们需要在一致性、失败容错、延迟和性能之间取得平衡，按照CAP定理，最终一致性性能很好，弱一致性在一些场合是可以接受的， 不管怎样，在实际中这种调和是非常复杂的，，有一套理论能够指导如何设计一个正确的乐观系统。CRDT是这样一个通向最终一致性的理论。

　　那么CRDT代表什么呢？在分布式系统中，复制途径有两种实现：一种是在主节点和从节点之间复制操作引起的状态结果，还有一种是就是复制操作自身，如果复制状态，会有一些状态收敛convergence规则，因此你可以创建Convergent Replicated Data Types；如果你复制的是操作，那么操作必须被仔细设计为commute ，你能创建Commutative Replicated Data Types，无论是 ‘convergent’和‘commutative’ 都是以C开头，所以，我们称为这两种都是CRDT。在这两种情况下，高级目标都是通过确保操作彼此不会冲突独立发生从而为了避免协调。也可以称它们为Conflict-free Replicated Data Type。

　　打个比喻，早期语言中有set list map这些标准的数据类型实现，然后我们就看到集合的并发版本等技术介绍，使用CRDT，我们看到了分布式集合等相关数据类型的诞生，最终，任何有自尊的语言或框架会拿出一套分布式集合库包，比如[Riak already supports CRDTs](http://docs.basho.com/riak/latest/theory/concepts/crdts/)和 [Jonas has an Akka CRDT library in github](https://github.com/jboner/akka-crdt)
　　
　　在理解CRDT之前，最好阅读因果一致性与COPS。

　　假设一个对象如Set有许多复制被分布，我们希望这些复制最终收敛converge到相同状态，我们希望在每个复制节点上查询这个对象的所有操作都能返回相同的结果，对于两个复制i和j，它们需要安全和活性条件：

- 安全Safety:如果i和j的因果历史是相同的，那么i和j的抽象状态是等效的。
- 活性Liveness: 如果一些事件e是i的因果历史，那么它最终将在j的因果历史中。
如果我们有任何i和j的成双最终收敛融合convergence，那么意味着复制对象的任何非空子集将会收敛，只要所有复制最终接受到所有更新。

###基于状态的CRDT

　　术语看起来很吓人，其实核心思想很简单，如果你理解max(x,y)，你就能理解基于状态的CRDT是如何工作的，让我们假设我们的状态是一个简单整数值，假设是4和6，那么最大Max是6，你能描述6作为两个值中最小上界，4和6的最小值是都是小于<=它，当然我们需要组合的状态值并不总是整数，但是只要我们能定义一个基于状态的有意义的最小上限least upper bound (LUB) 函数，我们就能创建CRDT，，你可能会听到反复提到的词语“ join semilattice”，这是使用基于LUB偏序函数的值的集合，比如整数集合与max。

　　假设一个对象的值来自于一个semilattice，并且定义了状态值的merge操作是一个最小上限函数，那么这样的对象的值将只会越来越大(因为是由最小上限函数返回的)，这是一个monotonic 单调的semilattice，那么这个类型是一个CRDT，这个类型的复制将是最终收敛的。
　　
###基于操作的CRDT

　　对于一个基于操作的对象，我们假设传递通道比如是TCP能传递更新操作，以一种数据类型比如因果传递的顺序传递，没有被这种顺序覆盖的操作被称为并发，如果所有这些并发都是commute，那么所有带有传递顺序的执行顺序一致性是等价的，所有复制将会收敛汇聚到相同状态，比如加法和减法是属于commute的，+7,-5与-5,+7会得到相同结果。

　　有趣的是，使用基于操作的方式总是能够模拟基于状态的方式。
　　
###CRDT案例

　　基于状态的计数器作为CRDT案例很典型，计数器的加减是属于commute，我们首先开始只增加的计数器案例，如果两个独立复制都增加了计数器，比如从0到1，那么我们会使用max最大值merge，最终我们得到了1，而不期望是2，让我们为每个复制节点在向量vector中在向量时钟后面保留一个条目作为更复杂的状态结构建模，增加一个指定复制节点会增加其向量中计数器，现在merge会取得每个条目的最大值，那么计数器值是所有条目的总和。

　　如果有减法的计数器将会更复杂，有兴趣者可进一步研究。下面看看CRDT的其他一些应用：

- Last-Writer-Win注册器(一个注册器是一个能够存储对象和值的单元) – 基于时间戳merge
- Multi-value 注册器 – 基于版本向量merge，如Amazon的Dynamo 
- Grow-only Set (支持新增和寻找). 这将会表明对于高级类型是有用的构建块。
- 2P-Set (two-phase两段集合set), 其每个条目能被增加或可选移除，但是不会在其后再次增加
- U-Set, (unique set). 2P-Set的简化。
- Last-Writer-Wins element Set, 元素条目是有时间戳，有add-set 和 remove-set
- PN-Set 为每个元素保留计数器。
- Observed-Remove Set。
- 2P2P-Graph
- add-only monotonic DAG
- add-and-remove partial order data type
一些支持协调文本编辑的数据类型


