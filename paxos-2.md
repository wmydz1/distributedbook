# Paxos算法2-算法过程


请先参考前文：Paxos算法1
 
1.编号处理
根据P2c ，proposer在提案前会先咨询acceptor查看其批准的最大的编号和value，再决定提交哪个value。之前我们一直强调更高编号的proposal，而没有说明低编号的proposal该怎么处理。 

|--------低编号(L<N)--------|--------当前编号(N)--------|--------高编号(H>N)--------|  

P2c 的正确性是由当前编号N而产生了一些更高编号H来保证的，更低编号L在之前某个时刻，可能也是符合P2c 的，但因为网络通信的不可靠，导致L被延迟到与H同时提交，L与H有可能有不同的value，这显然违背了P2c ，解决办法是acceptor不接受任何编号已过期的proposal，更精确的描述为：

####P1a ： An acceptor can accept a proposal numbered n iff it has not responded to a prepare request having a number greater than n.

显然，acceptor接收到的第一个proposal符合这个条件，也就是说P1a 蕴含了P1。
关于编号问题进一步的讨论请参考下节的【再论编号问题：唯一编号 】。

2. Paxos算法形成  

重新整理P2c 和P1a 可以提出Paxos算法，算法分2个阶段：
Phase1：prepare  

   （a）proposer选择一个proposal编号n，发送给acceptor中的一个多数派  
   
   （b）如果acceptor发现n是它已回复的请求中编号最大的，它会回复它已accept的最大的proposal和对应的value（如果有）；同时还附有一种承诺：不会批准编号小于n的proposal
   
####Phase2：accept  
 
   （a）如果proposer接收到了多数派的回应，它发送一个accept消息到（编号为n，value v的proposal）到acceptor的多数派（可以与prepare的多数派不同）  
       关键是这个value v是什么，如果acceptor回应中包含了value，则取其编号最大的那个，作为v；如果回应中不包含任何value，则有proposer随意选择一个  
       
   （b）acceptor接收到accept消息后check，如果没有比n大的回应比n大的proposal，则accept对应的value；否则拒绝或不回应
 
感觉算法过程异常地简单，而理解算法是怎么形成却非常困难。再仔细考虑下，这个算法又会产生更多的疑问：

####再论编号问题：唯一编号

保证paxos正确运行的一个重要因素就是proposal编号，编号之间要能比较大小/先后，如果是一个proposer很容易做到，如果是多个proposer同时提案，该如何处理？Lamport不关心这个问题，只是要求编号必须是全序的，但我们必须关心。这个问题看似简单，实际还稍微有点棘手，因为这本质上是也是一个分布式的问题。
 
在Google的Chubby论文中给出了这样一种方法：
假设有n个proposer，每个编号为ir (0<=ir <n)，proposol编号的任何值s都应该大于它已知的最大值，并且满足：s %n = ir => s = m*n + ir

proposer已知的最大值来自两部分：proposer自己对编号自增后的值和接收到acceptor的reject后所得到的值
以3个proposer P1、P2、P3为例，开始m=0,编号分别为0，1，2
P1提交的时候发现了P2已经提交，P2编号为1 > P1的0，因此P1重新计算编号：new P1 = 1*3+0 = 4
P3以编号2提交，发现小于P1的4，因此P3重新编号：new P3 = 1*3+2 = 5
 
整个paxos算法基本上就是围绕着proposal编号在进行：proposer忙于选择更大的编号提交proposal，acceptor则比较提交的proposal的编号是否已是最大，只要编号确定了，所对应的value也就确定了。所以说，在paxos算法中没有什么比proposal的编号更重要。

####活锁

当一proposer提交的poposal被拒绝时，可能是因为acceptor promise了更大编号的proposal，因此proposer提高编号继续提交。 如果2个proposer都发现自己的编号过低转而提出更高编号的proposal，会导致死循环，也称为活锁。

####Leader选举

活锁的问题在理论上的确存在，Lamport给出的解决办法是选举出一个proposer作leader，所有的proposal都通过leader来提交，当Leader宕机时马上再选举其他的Leader。
Leader之所以能解决这个问题，是因为其可以控制提交的进度，比如果之前的proposal没有结果，之后的proposal就等一等，不着急提高编号再次提交，相当于把一个分布式问题转化为一个单点问题，而单点的健壮性是靠选举机制保证。
 
问题貌似越来越复杂，因为又需要一个Leader选举算法，但Lamport在fast paxos中认为该问题比较简单，因为Leader选举失败不会对系统造成什么影响，因此这个问题他不想讨论。但是后来他又说，Fischer, Lynch, and Patterson的研究结果表明一个可靠的选举算法必须使用随机或超时（租赁）。
 
Paxos本来就是选举算法，能否用paxos来选举Leader呢？选举Leader是选举proposal的一部分，在选举leader时再用paxos是不是已经在递归使用paxos？存在称之为PaxosLease的paxos算法简化版可以完成leader的选举，像Keyspace、Libpaxos、Zookeeper、goole chubby等实现中都采用了该算法。关于PaxosLease，之后我们将会详细讨论。
虽然Lamport提到了随机和超时机制，但我个人认为更健壮和优雅的做法还是PaxosLease。

####Leader带来的困惑

Leader解决了活锁问题，但引入了一个疑问：  

既然有了Leader，那只要在Leader上设置一个Queue，所有的proposal便可以被全局编号，除了Leader是可以选举的，与Paxos算法1 提到的单点MQ非常相似。  

那是不是说，只要从多个MQ中选举出一个作为Master就等于实现了paxos算法？现在的MQ本身就支持Master-Master模式，难道饶了一圈，paxos就是双Master模式？
 
仅从编号来看，的确如此，只要选举出单个Master接收所有proposal，编号问题迎刃而解，实在无须再走acceptor的流程。但paxos算法要求无论发生什么错误，都能保证在每次选举中能选定一个value，并能被learn学习。比如leader、acceptor,learn都可能宕机，之后，还可能“苏醒”，这些过程都要保证算法的正确性。  

如果仅有一个Master，宕机时选举的结果根本就无法被learn学习, 也就是说，Leader选举机制更多的是保证异常情况下算法的正确性，虚惊一场，paxos原来不是Master-Master。  

在此，我们第一次提到了"learn"这个角色，在value被选择后，learn的工作就是去学习最终决议，学习也是算法的一部分，同样要在任何情况下保证正确性，后续的主要工作将围绕“learn”展开。

####Paxos与二段提交

Google的人曾说，其他分布式算法都是paxos的简化形式。 

假如leader只提交一个proposal给acceptor的简单情况： 

- 发送prepared给多数派acceptor
- 接收多数派的响应
- 发送accept给多数派使其批准对应的value
其实就是一个二段提交问题，整个paxos算法可以看作是多个交叉执行而又相互影响的二段提交算法

####如何选出多个Value

Paxos算法描述的过程是发生在“一次选举”的过程中，这个在前面也提到过，实际Paxos算法执行是一轮接一轮，每轮还有个专有称呼：instance（翻译成中文有点怪），每instance都选出一个唯一的value。
 
在每instanc中，一个proposal可能会被提交多次才能获得acceptor的批准，一般做法是，如果acceptor没有接受，那proposer就提高编号继续提交。如果acceptor还没有选择（多数派批准）一个value，proposer可以任意提交value，否则就必须提交意见选择的，这个在P2c 中已经说明过。
 
Paxos中还有一个要提一下的问题是，在prepare阶段提交的是proposal的编号，之后再决定提交哪个value，就是value与编号是分开提交的，这与我们的思维有点不一样。
 
####3. 学习决议  
 
在决议被最终选出后，最重要的事情就是让learn学习决议，学习决议就是决定如何处理决议。
在学习的过程中，遇到的第一个问题就是learn如何知道决议已被选出，简单的做法就是每个批准proposal的acceptor都告诉每个需要学习的learn，但这样的通信量非常大。简单的优化方式就是只告诉一个learn，让这个唯一learn通知其他learn，这样做的好是减少了通信量，但坏处同样明显，会形成单点；当然折中方案是告诉一小部分learn，复杂性是learn之间又会有分布式的问题。
无论如何，有一点是肯定的，就是每个acceptor都要向learn发送批准的消息，如果不是这样的话，learn就无法知道这个value是否是最终决议，因此优化的问题缩减为一个还是多个learn的问题。
 
能否像proposer的Leader一样为learn也选个Leader？因为每个acceptor都有持久存储，这样做是可以的，但会把系统搞的越来越复杂，之后我们还会详细讨论这个问题。
Learn学习决议时，还有一个重要的问题就是要按顺序学习，之前的选举算法花费很多精力就是为了给所有的proposal全局编号，目的是能被按顺序使用。但learn收到的决议的顺序可能不不一致，有可能先收到10号决议，但9号还未到，这时必须等9号到达，或主动向acceptor去请求9号决议，之后才能学习9号、10号决议。

####4. 异常情况、持久存储  

在算法执行的过程中会产生很多的异常情况，比如proposer宕机、acceptor在接收proposal后宕机，proposer接收消息后宕机，acceptor在accept后宕机，learn宕机等，甚至还有存储失败等诸多错误。  

但无论何种错误必须保证paxos算法的正确性，这就需要proposer、aceptor、learn都做能持久存储，以做到server”醒来“后仍能正确参与paxos处理。  

- propose该存储已提交的最大proposal编号、决议编号（instance id）
- acceptor储已promise的最大编号；已accept的最大编号和value、决议编号
- learn存储已学习过的决议和编号  

以上就是paxos算法的大概介绍，目的是对paxos算法有粗略了解，知道算法解决什么问题、算法的角色及怎么产生的，还有就是算法执行的过程、核心所在及对容错处理的要求。
但仅根据上面的描述还很难翻译成一个可执行的算法程序，因为还有无限多的问题需要解决：

- Leader选举算法
- Leader宕机，但新的Leader还未选出，对系统会有什么影响
- 更多交叉在一起的错误发生，还能否保证算法的正确性
- learn到达该怎么学习决议
- instance no、proposal no是该维护在哪里？
- 性能

众多问题如雪片般飞来，待这些都逐一解决后才能讨论实现的问题。当然还有一个最重要的问题，paxos算法被证明是正确的，但程序如何能被证明是正确的？

