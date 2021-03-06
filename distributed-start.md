#如何学习掌握一个分布式系统？

长期以来学习掌握分布式系统的知识非常庞杂混乱，本文将分布式算法归纳为几种：计时模型timing model; 进程间通讯interprocess communication 和失败模型failure model。

###计时模型timing model

计时模型分同步 异步和部分同步三种，这几种模型都有时间计时这个共同特点。

同步模型是直接调用执行，组件之间同时按步骤执行，这个模型的问题是无法反映现实情况，甚至在分布式情况下很少有真正同步，比如过去RPC(远程过程调用)等都是两个服务器之间的代码方法直接相互调用，这种问题带来相互堵塞各种服务器进程，现在服务器之间都是通过发送消息实现通讯，让发送消息变成同步几乎很难。同步模型好处是能完成理论上测试结果，比如，因为同步模型有时间上的保证，我们可以看看一个问题在同步模型下是否能够解决，如果在有时间保证的机制下都不能解决，意味着在没有时间保证的机制也是不可能解决的。

异步模型有点复杂，组件之间的动作是按照它们自己的顺序要求进行的，也不提供任何关于采取这些行动的时间上与速度的保证，这个模型更接近于现实情况，但是也不是完美的，比如一个进程会需要无限循环来响应一个请求，在真实项目中，我们可能会强加一个计时timeout，一旦超过这个timeout，将会退出这个请求的处理，这就带来了问题，如果确保一个进程活跃的条件？也就是说，如何知道一个进程是需要无限循环活跃的，而其他进程则是不需要，需要timeout去中断的，这里面哪个是业务需要，哪个是因为故障导致的呢？

在部分同步模型中大部分访问同步时钟，有关于传递消息有多长的限制，有一个进程执行一个步骤需要多长时间的限制。

###进程间通讯

进程之间是如何通讯的，这里有消息传递模型和共享内存模型，前者是通过消息发送通讯，后者是访问内存中共享变量共享数据进行通讯。这里进程有服务器 节点的意思，一个进程可能代表分布式场景的一台服务器。

消息传递最难的是不能发送重复消息，每次只能精确一次传递，这里有很多设计，比如Perfect Links 抽象可以保证，但是它不能正常反映现实世界，虽然不真实，但是有用，我们可以使用Perfect Links 证明一个问题不可能被解决，然后我们就知道其他相关问题也没有答案。消息传递总是可以被想象为FIFO之类队列或堆栈。

共享内存是我们编程常用的方式，需要在一台服务器内才能完成。我们可以使用消息传递算法完成分布式情况下的内存共享对象，比如读写注册器，调用一个服务之间需要查询这个服务在哪个服务器上，负载平衡器也是一个读写注册器，是一个全局共享的内存。

###失败模型
分布式模型总是必须考虑进程失败的情况，在crash-stop失败模型中，一个进程假设为一直是正确，直至它崩溃，一旦它崩溃，就永远不会恢复；也有crash-recovery 模型，进程能够在失败以后恢复，在这种情况下，一些算法来保证进程恢复到其失败之前的状态，这可以通过从持久层读取状态完成，或者通过和一个集群小组中的其他进程通讯方式完成。注意这里有不同集群组算法，一个进程崩溃后，恢复其状态的进程不会再被认为是之前同样的进程，这取决于动态组还是固定组这两种算法。

失败模型也包括：一个进程如果无法接受和发送消息，被称为遗漏omission failure mode，遗漏模型也有不同种类，一个进程无法接受和发送消息很重要吗？想象一组进程实现一个分布式缓存，如果一个进程无法回复同一组的其他进程，即使能够接受来自它们的请求，这也意味着这个进程能够接受外部消息更新自己的状态，其实也就意味着它能回复来自客户端的读请求，也就是说，虽然它自己不能主动回复客户端的请求，但是可以接受客户端的主动读取请求。

一个复杂失败模型是拜占庭Byzantine 或称为任意失败模型，进程会发送错误信息到对方，它们会模仿发送正确数据，但是实际已经篡改了本地数据库的内容。

设计分布式系统时，我们需要对付这些失败模型。

###失败探测
我们希望在进程崩溃失败时及时发现，比如crash-stop失败模型加上同步系统，我们能够使用timeout；如果我们定期让进程ping到一个专门的失败探测器，我们就能精确知道那个进程是否正常，如果过了timeout时间没有Ping访问，那么我们就可以认为那台进程服务器崩溃了。

更真实情况是，假设一个消息到达目标需要确定的时间，确定好一个进程执行一个步骤需要多长时间，那么就可以使用timeout进行衡量计算。

失败模型探测有两个属性策略：
1. Strong Completeness强完整性：每个失败的进程会永久被其他正确进程怀疑。
2.Eventual Strong Accuracy最终强精确度，没有一个进程被任何正确的进程怀疑。

当一个进程被其他进程怀疑时，这些进程就不可能达成共识consensus ，而在分布式系统中使用异步模型是必须要达成共识，也就是每个进程内部状态通过异步消息传递后，最终其他进程的状态会和最初发送消息的那个进程内部状态一致，这称为达成共识，但是因为有进程存在失败崩溃的可能，所以，在这个达成共识的消息传递过程中，如何确保进程之间的信任，不怀疑对方，从而确保消息传递成功，那么引入失败探测器是可以规避这个问题的。

领导人选举LEADER ELECTION
这是通过决定某个进程没有崩溃失败，能够正常工作，那么这个进程就可以被网络中其他进程信任，它就可以被认为是领导人，负责协调分布式动作，这种协议有Raft和Zab两种。

这种机制会导致瓶颈集中在领导人那里，而且之前还需要领导人选举，这些多余过程可能是我们不需要的。

###一致共识CONSENSUS
共识是在独立进程之间达成一致的统一意见，这些进程会就某个问题建议一个数值，基于这个推荐的值会同意采取一致行动，比如，一个轿车有各种传感器提供制动器温度的信息，依赖于传感器的精度，会有不同变化的数值，但是ABS计算机需要知道施加多大压力到制动器上，这种共识问题每天生活中都发生。

一个进程实现共识是通过暴露带有推荐和决定功能的API实现的，一个进程会推荐数值，由此开始共识，然后它得基于一个数值决定，这个数值是在整个系统中被推荐了的，这些算法包括：Termination, Validity, Integrity 和Agreement.
1.Termination: 每个正确的进程最终会决定某个数值。
2.Validity: 如果一个进程决定了v，那么v会被其他进程推荐。
3.Integrity: 没有进程能够决定两次
4.Agreement: 没有两个正确进程有不同的决定。

###法定人数QUORUMS
Quorums 是一个设计失败容错分布式系统的工具，当系统存在crash-failure模型时，总是有一个法定人数代表大多数意见从而进行决策的，因为崩溃失败的总是少数。

比如有N个进程服务器，假设崩溃的进程是少数，比如N/2-1个进程崩溃，也就是49%的进程崩溃，我们还是有51%的会投赞成票。Raft协议使用的是这种大多数策略，根据提交到系统的日志来判断，

###分布式系统的时间
理解时间和其导致的因果是分布式系统的大问题，我们通常用事件这个概念代表生活中发生的那些事实，使用happened before顺序约束定义这些事件，但是我们有很多进程交换信息，共同访问共享资源等等，我们如何告诉某个进程事件的happened before策略呢？也就是谁在前谁在后的顺序呢？为了回答这个问题，进程需要共享一个同步的时钟，精确知道它在网络间移动花费多长时间？包括CPU调度任务的时间等等，显然这在真实世界是不可能实现的。

Time, Clocks, and the Ordering of Events in a Distributed System引入了逻辑时钟概念，逻辑时钟是一个分配一个数字给事件的方式，也就是说，这些数字不是和实际时间有关，但是和一个节点的进程事件有关。

有各种逻辑时钟，比如 Vector Clocks向量时钟或 Interval Tree Clocks.

理解分布式时间问题，必须理解一个重要概念：同时性这个想法有时我们必须放弃（The idea of simultaneity is something we have to let go.），这是有关“绝对知识”等旧哲学信条的问题，他们认为绝对知识是可以到达的，其实人的认识是相对，永远不可能到达真正事物本质，你以为的同时性并不是真正同时性，光线也是有速度的，即使最快的光线也是需要时间才从一个地方到达另外一个地方。可见Inventing the Enemy发明敌人 一书中的“绝对与相对”。

[What We Talk About When We Talk About Distributed](http://videlalvaro.github.io/2015/12/learning-about-distributed-systems.html#a-quick-look-at-flp)