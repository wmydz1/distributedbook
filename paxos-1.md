Paxos算法的难理解与算法的知名度一样令人敬仰，从我个人的经历而言，难理解的原因并不是该算法高深到大家智商不够，而在于Lamport在表达该算法时过于晦涩且缺乏一个完整的应用场景。如果大师能换种思路表达该算法，大家可能会更容易接受：

- 首先提出算法适用的场景，给出一个多数读者能理解的案例
- 其次描述Paxos算法如何解决这个问题
- 再次给出算法的起源（就是那些希腊城邦的比喻和算法过程）

Lamport首先提出算法的起源，在没有任何辅助场景下，已经让很多人陷于泥潭，在满脑子疑问的前提下，根本无法继续接触算法的具体内容，更无从体会算法的精华。本文将换种表达方法对Paxos算法进行重新描述。
我们所有的描述都假设读者已经熟读了Lamport的paxos-simple一文，因此对各种概念不再解释。
除了Lamport的几篇论文，对Paxos算法描述比较简洁的中文文章是：http://zh.wikipedia.org/zh-cn/Paxos%E7%AE%97%E6%B3%95，该文翻译的比较到位，但在关键细节上还是存在一些歧义和一些对原文不正确的理解，可能会导致读者对Paxos算法更迷茫，但阅读该文可以快速地对Paxos算法有个大概的了解。
 
####1.应用场景
 
（1）分布式中的一致性

- Paxos算法主要是解决一致性问题，关于“一致性”，在不同的场景有不同的解释：
- NoSQL领域：一致性更强调“能读到新写入的”，就是读写一致性
- 数据库领域：一致性强调“所有的数据状态一致”，经过一个事务后，如果事务成功，所有的表数据都按照事务中的SQL进行了操作，该修改的修改，该增加的增加，该删除的删除，不能该修改的修改了，该删除的没删掉；如果事务失败，所有的数据还是在初始状态；
 
- 状态机：在状态机中的一致性更强调在每个初始状态一致的状态机上执行一串命令后状态都必须相互一致，也就是顺序一致性。Paxos算法中的一致性指的就是这种情况，接下来我们会对这种场景进一步讨论。 

（2）MQ
假如所有系统的Log信息都写入一个MQ Server，然后通过MQ把每条Log指令发异步送到多个Log Server写入文件（写入多个Log Server的原因是对Log文件做备份以防数据丢失），则所有Log Server上的数据肯定是一致的（Log内容及顺序完全相同），因为MQ本身就有排序功能，只要进了Q数据也就有了序，相当于编了全局唯一的号，无论把这些数据写入多少个文件，只要按编号，各文件的内容必定是一致的，但一个MQ Server显然是一个单点，如果宕机，会影响整个系统的可用性。
 
（3）多MQ
要解决MQ单点问题，首选方案是采用多个MQ Server，即使用一个MQ Cluster，客户端可以访问任意MQ Server，不同的客户端可能访问不同MQ Server，不同MQ Server上的数据内容、顺序可能不一致，如果不解决这个问题，每个MQ Server写入Log Server的内容就不一致，这显然不是我们期望的结果。
 
（4）NoSQL中的数据更新
一般的NoSQL都会通过数据复制的形式保证其可用性，但客户端对多数据进行操作时，可能会有很多对同一数据的操作发送的某一台或几台Server，有可能执行：Insert、Update A、Update B....Update N，就一次Insert连续多次Update，最终复制Server上也必须执行这一的更新操作，如果因为线程池、网络、Server资源等原因导致各复制Server接收到的更新顺序不一致，这样的复制数据就失去了意义，如果在金融领域甚至会造成严重的后果。
 
上面这些不一致问题性正是Paxos算法要解决的，当然这些问题也不是只有Paxos能解决，在没有Paxos之前这些问题也得到了解决，比如通过使用双Master模式的MQ解决MQ单点问题；通过使用Master Server解决NoSQL的复制问题，但这些解决方法都存在一些缺陷，要么难水平扩展，要么影响可用性。当然除了Paxos算法还有其他一些算法也试图解决这类问题，比如：Viewstamped Replication算法。
 
上面描述的这些场景的共性是希望多Server之间状态一致，也就是一致性，再看中文Wiki开篇提到的：

> 在一个分布式数据库系统中，如果各节点的初始状态一致，每个节点都执行相同的操作序列，那么他们最后能得到一个一致的状态。为保证每个节点执行相同的命令序列，需要在每一条指令上执行一个“一致性算法”以保证每个节点看到的指令一致

大家或许会对该描述有更深的理解。
 
####2.Paxos如何解决这类问题
Paxos对这类问题的解决就是试图对各Server上的状态进行全局编号，如果能编号成功，那么所有操作都按照编号顺序执行，一致性就不言而喻。当Cluster中的Server都接收了一些数据，如何进行编号？就是表决，让所有的Server进行表决，看哪个Server上的哪个数据应该排第一，哪个排第二...，只要多数Server同意某个数据该排第几，那就排第几。

很显然，为了给每个数据唯一编号，每次表决只能产生一个数据，否则表决就没有任何意义。Paxos的算法的所有精力都放在如何在一次表决只产生一个数据。再进一步，我们称表决的数据叫Value，Paxos算法的核心和精华就是确保每次表决只产生一个Value。
 
###3.Paxos算法
我们对原文的概念加以补充：  

- promise：Acceptor对proposer承诺，如果没有更大编号的proposal会accept它提交的proposal
- accept：Acceptor没有发现有比之前的proposal更大编号的proposal，就批准了该proposal
- chosen：当Acceptor的多数派都accept一个proposal时，该proposal就被最终选择，也称为决议
也就是说，Acceptor对proposer有两个动作：promise和accept 

下面的解释也主要围绕着”Only a single value is chosen,“，再看下条件P1，
 
###P1：An acceptor must accept the first proposal that it receives.

乍一看，这个条件是显然的，因为之前没有任何value，acceptor理所当然地应该accept第一个proposal，但仔细想想，感觉P1这个条件很不严格，到底是一个对问题的简单描述还是一个数学上严格的必要条件？这些疑问归结为2个问题： 

（1）这个条件本质上在保证什么？  
（2）第二个proposal怎么办？
 
在后续的算法中看到一个Acceptor是否批准一个Value与它是否是第一个没有任何关系，而与这个Proposal的编号有关。那岂不说明P1没有得到保证？开始我也百思不得其解，后来经过跟朋友交流发现，P1中的"accept"其实是指acceptor对proposer的"promise"，就是语言描述跟算法的步骤描述之间存在歧义，因此我认为对算法问题还是应该采用数学语法而非文字语言。
 
所以，P1是强调了第一个proposal要被promise，但第二个还未提到，这也是疑问之一。
也很显然的是，单靠P1是无法保证Paxos算法的，因可能无法形成多数派，那接下来的讨论应该是考虑如何弥补P1的缺点，使其可以保证Paxos算法，就是我们希望未来的条件应该说明：

- 如何解决P1中无法形成多数派的问题
- 第二个proposal如何选择

于是约束P2出现了：

####P2：If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v.

P2的出现让人大跌眼镜，P2并没沿着P1的路向下走，也没有解决P1的上述2个不完备，而是从另一个侧面讨论如何保证只能选出一个Value。P1讨论的是该如何选择，P2讨论的是一旦被选出来，之后的选择应该不变，就是P1还在讨论选的问题，P2已经选出来了，中间有个断层，怎么选的没有讨论。 

其实从后面Lamport不断对P2增强可以看出，P2里面蕴含着P1（通过proposal编号，第一次之前没有编号，所以选择），P2才真正给出了怎么选择的具体过程，从事后分析看，P1给出了第一个该怎么选，P2给出了所有的该怎么选，条件有点重复。所以，把P1和P2看作是两个独立条件的做法是不准确的，因而中文wiki中提到“如果 P1 和 P2 都能够保证，那么约束2就能够保证”，对细微理解有一定的影响。 

也不是说P1就没有用，反过来看，P2是个未知问题，而P1是这个未知问题的已知部分，从契约的角度来看，P1就是个不变式，任何对P2的增强都不能过了头以至于无法满足P1这个不变式，也就是说，P1是P2增强的底线。  

那还有没有其他的不变式需要遵守？是否在对P2增强的过程中已破坏了这些未知的不变式？这些高难度的问题牵扯到Paxos算法正确性，要看MIT的严格的数学证明，已超出了本文。  

另外，中文Wiki对P2的描述是：“P2：一旦一个 value 被批准(chosen)，那么之后批准(chosen)的 value 必须和这个 value 一样。”，原文采用higher-numbered更能描述未来对proposal进行编号这个事实，而中文采用“之后”，已经完全失去这个意义。
 
我们暂时按下P1不表，近距离观察一下P2，为了保证每次选出一个value，P2规定在一个Value已经被选出的情况下，如果还有其他的proposer提交value，那之后批准的value应该跟前一个一致，就是在事实上已经选定一个value时，之后的proposer不能提交不同的value把之前的结果打乱。这是一个泛泛的描述，但如果这个描述能得到实现，paxos算法就能得到保证，因此P2也称"safety property"。
接下来的讨论都时基于“If a proposal with value v is chosen”，如何保证“then every higher-numbered proposal that is chosen has value v”，具体怎么做到“a proposal with value v is chosen"暂且不谈。
 
P2更多是从思想层面上提出来该如何解决这个问题，但具体的落实工作需要很多细化的步骤，Lamport是通过逐步增强条件的方式进行落实P2，主要从下面几个方面进行：

- 对整个结果提出要求（P2）
- 对Acceptor提出要求（P2a）
- 对Proposer提出要求（P2b）
- 对Acceptor与Proposer同时提出要求（P2c） 

Lamport为什么能把过程划分的如此清楚已经不得而知，但从Lamport发表的文章来看，他对分布式有很深的造诣，也持续了很长的时间，能有如此的结果，与他对分布式的基础与背后的巨大努力有很大关系。但对我们而言，不知过程只知个结果，总感觉知其然不知其所以然。
我们沿着上面的思路继续： 

####P2a：If a proposal with value v is chosen, then every higher-numbered proposal accepted by any acceptor has value v.

这个条件是在限制acceptor，很显然，如果P2a得到了满足，满足P2是肯定的，但P2a的增强破坏了P1不变式的底线，具体参考原文，所以P2a本身没啥意义，转而从proposer端进行增强。
 
####P2b：If a proposal with value v is chosen, then every higher-numbered proposal issued by any proposer has value v.

这个条件是在限制proposer，如果能限制住proposer，对acceptor的限制当然能被满足的。同时，因为限制proposer必须提交value v，也就顺便保证了P1（第一个肯定是value v）
但P2b是难以实现的，难实现的原因是多个proposer可以提交任意value的proposal，无法限制proposer不能提交某个value，因此需要寻找P2b的等价条件：
 
####P2c：For any v and n, if a proposal with value v and number n is issued, then there is a set S consisting of a majority of acceptors such that either
(a) no acceptor in S has accepted any proposal numbered less than n, or
(b) v is the value of the highest-numbered proposal among all proposals numbered less than n accepted by the acceptors in S.  

根据原文，P2c里面蕴含了P2b，但由P2c推导P2b是最难理解的部分。
首先要清楚P2c要做什么，因为P2b很难直接实现，P2c要做的就是解决P2b的问题，就是解决“如果value v被选择了，更高编号的提案已经具有value v”，也就是说：  

- R：“For any v and n, if a proposal with value v and number n is issued”是结果，而
- C：“ then there is a set S consisting...”是条件 

就是要证明如果C成立，那么结果R成立，而原文的表达是“如果R成立，那么存在一个条件R”，容易让人搞混因果关系，再次感叹如果使用数学符号表达这样的歧义肯定会减少很多。
P2c解决问题的思路是：不是直接尝试去满足P2b，而是寻找能满足P2b的一个充分条件，如果能满足这个充分条件，那P2b的满足是显然的。还要强调一点的是proposer可以提交任意的value，你怎么能限制我提交的必须是value v呢？其实原文中的“For any v and n, if a proposal with value v and number n is issued”是指“如果一个编号为n的proposal提交value v，并且value v能被acceptor所接受”，要想被接受就不能随便提交一个value，就必须是一个受限制的value，这里讨论的前提是value v是要被接受的。然后我们再看下，是否满足了条件C，结果R就成立。
 
####(a) no acceptor in S has accepted any proposal numbered less than n
如果这个条件成立，那么n是S中第一个proposal，根据P1，必须接受，所以结果R成立
 
####(b) v is the value of the highest-numbered proposal among all proposals numbered less than n accepted by the acceptors in S  

这个证明先假设编号为n的proposal具有value X被选择，肯定存在一个集合C，其中的每个acceptor都接受了value X，而集合S中的每个Acceptor都接受了value v，因为S、C都是多数派，所以存在一个公共成员u，既接受了X，又接受了v,为了保证选择的唯一性，必须X=v.
大家可能会发觉该证明有点不太严格，“小于n的最大编号”与n之间还有很多proposal，那些proposal也有一些value，那些value会不会不是v？
这个就会用到原文中的数学归纳法，就是任意的编号m的proposal具有了value v，那么n=m+1是，根据上面也是具有value v的，那么向后递推，任意的n >m都具有value v。中文wiki中的那个归纳证明不需要对m...n-1正推，而对n反证，通过数学归纳正推完全可以得出最终结果。
 
也就是说，P2c是P2b的一个加强，满足P2c就能满足P2b。
我们再近距离观察下P2c，发现只要在proposer提交提案前，咨询一下acceptor，看他们的最高编号是啥，他们是否选择了某个value v，再根据acceptor的回答进行选择新的编号、value提交，就可以满足P2c。通过编号，可以把(a)和(b)两个条件统一在一起。
 
其实P2c要表达的思想非常简单：如果前面有value v选出了，那以后就提交这个value v；否则proposer决定提交哪个value，具体做法就是事前咨询，事中决定，事后提交，也就是说可以通过消息传递模型实现。Lamport通过条件、集合、归纳证明等形式表达该问题，而没提这样做的目的，会导致理解很困难。大家可能会比较疑惑，难道自始至终只能选出一个value？其实这里的选出，是指一次选举，而不是整个选举周期，可以多次运行paxos，每次都只选出一个value。
 
满足P2c从侧面也反映出要想提交一个正确的value v，要对proposer、acceptor同时进行限制，仅限制一方问题是无法解决的。
再回顾下条件之间的递推关系P2c=>P2b=>P2a=>P2，就是说P2c最终保证了P2，也就是解决了如何做到一个value v被选择之后，被选择的编号更大的proposal都具有value v，P2c不仅保证P2的结果，更提出了“如何选”的问题，就是上面分阶段进行，这就填补了P1与P2之间缺少如何选的断层，还有P1的2个不完备问题从直观上感觉会得到解决，具体的要看算法过程章节。
 
####P1的不完备问题：
P2c也顺便解决了P1的不完备问题，因为proposer提交的value是受acceptor限制的，就不会在一次选举中提交两个不同的value，即使能提交也会因为proposal编号问题有一个会被拒绝，从而能保证能形成多数派。
另一个关于第二个该怎么选的不完备问题，也是显然的了。
 
再次证明了，P2里面蕴含了P1，P1只是未知问题P2的不变式。