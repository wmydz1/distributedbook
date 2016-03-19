#分布式网络是否可靠?

来自[The network is reliable](http://aphyr.com/posts/288-the-network-is-reliable)一文阐述当前发生在业界的一场有关分布式系统是否可靠的讨论。

正方观点是网络是可靠的，没有必要那么专注于失败恢复设计，而另外一方认为网络分区应该尽可能少，它引起的问题比预期要多。
这场讨论将从根本上影响分布式数据库、队列、应用程序的走向。到底谁是正确的？

最大问题是缺乏证据，目前我们所知道的分布式网络基本来自技术人员喝啤酒时的饭后茶余的谈资，大部分来自猜测和传言，不真实。

下面搜集来自各大著名网络中心的数据：
一队来自加拿大多伦多大学和微软研究院的研究：有关在一些微软的数据中心网络出现故障的行为 。 他们发现，平均每天5.2个设备和40.8个连接数的故障率，平均修复时间约5分钟（一个星期）。 虽然研究人员注意到，相关的链路故障和通信分区是具有挑战性的，他们估计每次故障平均丢包59,000包。 也许更担心的是他们发现，网络冗余只能提高43％的流量，网络冗余并不能消除许多常见的网络故障。

在美国加州大学，圣迭戈和惠普实验室的研究人员之间的一项联合研究投票，有关惠普的管理网络的网络故障的原因和严重程度： “连接问题”相关的投票占11.4％的支持票（14％为最高优先级），最高优先级的时间为2小时45分钟，平均时间为4小时18分钟。

谷歌小胖是谷歌的分布式锁管理器，在700天中超过61天停运，。 九次停运大于30秒，四次是网络维护，两次是由于“可疑的网络连接问题造成的。”

.....

原文还列举了亚马逊和雅虎和GitHub的大型网络问题，总结出分布式系统是很难的。

然后详细研究了PostgreSQL, Redis, MongoDB 和 Riak等有关分区冗余方面技术细节，有兴趣者可查看原文。

结论：
Distributed state is a difficult problem, but with a little work we can make our systems significantly more reliable。

分布式状态是很难的课题，但是付出一小步努力我们能使我们的系统更加可靠一些。

Consistency is a property of the data, not of the nodes. Avoid systems which assume node state consensus implies data consistency.

必须明白，一致性是数据的属性，不是服务器节点的，避免服务器节点一致性的思维影响数据一致性。

Even well-known algorithms like two-phase commit have some caveats, like false negatives. SQL transactional consistency comes in several levels. If you use the stronger consistency levels, remember that conflict handling is essential.
即使最著名的两段确认2PC算法也有失败的时候，SQL事务一致性有几个层次，如果你使用高一致性，一定要记住冲突处理是必要的。


Wall clocks are only useful for ensuring responsiveness in the face of deadlock, 挂钟仅仅对已死锁响应性是有用的，在这些测试中，所有的时钟与NTP同步，我们仍然丢失数据。

要认识到正确的算法和真正软件之间有巨大鸿沟，特别是在考虑延迟性这个指标时。

在性能和正确性之间的权衡，划分出需要高一致性的需求范畴，其他部分可以牺牲线性顺序化保证正确性。

不变性是一个非常有用的特性，并可结合强大的混合动力系统：一个可变的CP(CAP定理的CP)数据存储(不变和可变分离，有态和无态分离 banq注)。因为认识到业务上不变性，就可以利用尽可能多的幂等操作：使用队列和重试。更进一步充分利用CRDTs。

要意识到客户端(浏览器)也是分布式系统的一个重要组成部分。网络错误的意思是“我不知道，”而不是“失败”，考虑到你的系统边界延伸到客户端的一致性算法：例如使用TCP客户端ETags或向量时钟，或延长CRDTs到浏览器。
