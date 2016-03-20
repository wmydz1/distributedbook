#分布式一致性协议Raft原理与实例

1.Raft协议

1.1 Raft简介

Raft是由Stanford提出的一种更易理解的一致性算法，意在取代目前广为使用的Paxos算法。目前，在各种主流语言中都有了一些开源实现，比如本文中将使用的基于JGroups的Raft协议实现。关于Raft的原理，强烈推荐[动画版Raft讲解](http://thesecretlivesofdata.com/raft/)。

1.2 Raft原理

在Raft中，每个结点会处于下面三种状态中的一种：

- follower：所有结点都以follower的状态开始。如果没收到leader消息则会变成- 
- candidate状态
candidate：会向其他结点“拉选票”，如果得到大部分的票则成为leader。这个过程就叫做

- Leader选举(Leader Election)
leader：所有对系统的修改都会先经过leader。每个修改都会写一条日志(log entry)。leader收到修改请求后的过程如下，这个过程叫做日志复制(Log Replication)： 
复制日志到所有follower结点(replicate entry)
大部分结点响应时才提交日志
通知所有follower结点日志已提交
所有follower也提交日志
现在整个系统处于一致的状态

###1.2.1 Leader Election

当follower在选举超时时间(election timeout)内未收到leader的心跳消息(append entries)，则变成candidate状态。为了避免选举冲突，这个超时时间是一个150~300ms之间的随机数。

成为candidate的结点发起新的选举期(election term)去“拉选票”：

- 重置自己的计时器
- 投自己一票
- 发送 Request Vote消息

如果接收结点在新term内没有投过票那它就会投给此candidate，并重置它自己的选举超时时间。candidate拉到大部分选票就会成为leader，并定时发送心跳——Append Entries消息，去重置各个follower的计时器。当前Term会继续直到某个follower接收不到心跳并成为candidate。

如果不巧两个结点同时成为candidate都去“拉票”怎么办？这时会发生Splite Vote情况。两个结点可能都拉到了同样多的选票，难分胜负，选举失败，本term没有leader。之后又有计时器超时的follower会变成candidate，将term加一并开始新一轮的投票。

###1.2.2 Log Replication

当发生改变时，leader会复制日志给follower结点，这也是通过Append Entries心跳消息完成的。前面已经列举了Log Replication的过程，这里就不重复了。

Raft能够正确地处理网络分区（“脑裂”）问题。假设A~E五个结点，B是leader。如果发生“脑裂”，A、B成为一个子分区，C、D、E成为一个子分区。此时C、D、E会发生选举，选出C作为新term的leader。这样我们在两个子分区内就有了不同term的两个leader。这时如果有客户端写A时，因为B无法复制日志到大部分follower所以日志处于uncommitted未提交状态。而同时另一个客户端对C的写操作却能够正确完成，因为C是新的leader，它只知道D和E。

当网络通信恢复，B能够发送心跳给C、D、E了，却发现“改朝换代”了，因为C的term值更大，所以B自动降格为follower。然后A和B都回滚未提交的日志，并从新leader那里复制最新的日志。但这样是不是就会丢失更新？

##2.JGroups-raft介绍

2.1 JGroups中的Raft

JGroups是Java里比较流行的网络通信框架，近期顺应潮流，它也推出了Raft基于JGroups的实现。简单试用了一下，还比较容易上手，底层Raft的内部机制都被API屏蔽掉了。下面就通过一个分布式计数器的实例来学习一下Raft协议在JGroups中的实际用法。

Maven依赖如下：

```
    <dependency>
        <groupId>org.jgroups</groupId>
        <artifactId>jgroups-raft</artifactId>
        <version>0.2</version>
    </dependency>
    
 ```

其实JGroups-raft的Jar包中已经自带了一个[Counter的Demo](https://github.com/belaban/jgroups-raft/blob/master/src/org/jgroups/raft/demos/CounterServiceDemo.java)，但仔细看了一下，有的地方写的有些麻烦，不太容易把握住Raft这根主线。所以这里就参照官方的例子，进行了简写，突出Raft协议的基本使用方法。JGroups-raft目前资料不多，InfoQ上的[这篇文章](http://www.infoq.com/articles/JGroups-raft-Primer)很不错，还有[官方文档](http://belaban.github.io/jgroups-raft/manual/index.html)。

2.2 核心API

使用JGroups-raft时，我们一般会实现两个接口：RAFT.RoleChange和StateMachine：

- 实现RAFT.RoleChange接口的方法能通知我们当前哪个结点是leader
- 实现StateMachine执行要实现一致性的操作
典型单点服务实现方式就是：

```
JChannel ch = null;
RaftHandle handle = new RaftHandle(ch, this);
handle.addRoleListener(role -> {
    if(role == Role.Leader)
        // start singleton services
    else
        // stop singleton services
});

```

###2.3 默认配置

jgroups-raft.jar中已经带了一个raft.xml配置文件，作为实例程序我们可以直接使用它。

简要解释一下最核心的几个配置项，参照GitHub上的文档：

UDP：IP多播配置

- raft.NO_DUPES：是否检测新加入结点的ID与老结点有重复
- raft.ELECTION：选举超时时间的随机化范围
- raft.RAFT：所有Raft集群的成员必须在这里声明，也可以在运行时通过addServer/
- removeServer动态修改

- raft.REDIRECT：是否转发请求给leader
- raft.CLIENT：在哪个IP和端口上接收客户端请求

```



<config xmlns="urn:org:jgroups"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:org:jgroups http://www.jgroups.org/schema/jgroups.xsd">
    <UDP
         mcast_addr="228.5.5.5"
         mcast_port="${jgroups.udp.mcast_port:45588}"
         ... />
    ...
    <raft.NO_DUPES/>
    <raft.ELECTION election_min_interval="100" election_max_interval="500"/>
    <raft.RAFT members="A,B,C" raft_id="${raft_id:undefined}"/>
    <raft.REDIRECT/>
    <raft.CLIENT bind_addr="0.0.0.0" />
</config>

```

###3.JGroups-raft实例

实例很简单，只有JGroupsRaftTest和CounterService两个类组成。JGroupsRaftTest是测试启动类，而CounterService就是利用Raft协议实现的分布式计数服务类。

JGroupsRaftTest的职责主要有三个：

- 创建Raft协议的JChannel
- 创建CounterService
- 循环读取用户输入

目前简单实现了几种操作包括：初始化计数器、加一、减一、读取计数器、查看Raft日志、做Raft快照（用于压缩日志文件）等。其中对计数器的操作，因为要与其他Raft成员进行分布式通信，所以当前集群必须要多于一个结点时才能进行操作。如果要支持单结点时的操作，需要做特殊处理。

```
import org.jgroups.JChannel;
import org.jgroups.protocols.raft.RAFT;
import org.jgroups.util.Util;

/**
 * Test jgroups raft algorithm implementation.
 */
public class JGroupsRaftTest {

    private static final String CLUSTER_NAME = "ctr-cluster";
    private static final String COUNTER_NAME = "counter";
    private static final String RAFT_XML = "raft.xml";

    public static void main(String[] args) throws Exception {
        JChannel ch = new JChannel(RAFT_XML).name(args[0]);
        CounterService counter = new CounterService(ch);

        try {
            doConnect(ch, CLUSTER_NAME);
            doLoop(ch, counter);
        } finally {
            Util.close(ch);
        }
    }

    private static void doConnect(JChannel ch, String clusterName) throws Exception {
        ch.connect(clusterName);
    }

    private static void doLoop(JChannel ch, CounterService counter) {
        boolean looping = true;
        while (looping) {
            int key = Util.keyPress("\n[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit\n" +
                    "first-applied=" + ((RAFT) ch.getProtocolStack().findProtocol(RAFT.class)).log().firstApplied() +
                    ", last-applied=" + counter.lastApplied() +
                    ", commit-index=" + counter.commitIndex() +
                    ", log size=" + Util.printBytes(counter.logSize()) + ": ");

            if ((key == '0' || key == '1' || key == '2') && !counter.isLeaderExist()) {
                System.out.println("Cannot perform cause there is no leader by now");
                continue;
            }

            long val;
            switch (key) {
                case '0':
                    counter.getOrCreateCounter(COUNTER_NAME, 1L);
                    break;
                case '1':
                    val = counter.incrementAndGet(COUNTER_NAME);
                    System.out.printf("%s: %s\n", COUNTER_NAME, val);
                    break;
                case '2':
                    val = counter.decrementAndGet(COUNTER_NAME);
                    System.out.printf("%s: %s\n", COUNTER_NAME, val);
                    break;
                case '3':
                    counter.dumpLog();
                    break;
                case '4':
                    counter.snapshot();
                    break;
                case 'x':
                    looping = false;
                    break;
                case '\n':
                    System.out.println(COUNTER_NAME + ": " + counter.get(COUNTER_NAME) + "\n");
                    break;
            }
        }
    }

}

```

##3.2 CounterService

CounterService是我们的核心类，利用Raft实现了分布式的计数器操作，它的API主要由四部分组成：

Raft Local API：操作本地Raft的状态，像日志大小、做快照等  
Raft API：实现Raft的监听器和状态机的方法  
 
- roleChanged：本地Raft的角色发生变化
- apply：分布式通信消息
- readContentFrom/writeContentTo：读写快照

Counter API：计数器的分布式API

Counter Native API：计数器的本地API。直接使用的话相当于脏读

```
import org.jgroups.Channel;
import org.jgroups.protocols.raft.RAFT;
import org.jgroups.protocols.raft.Role;
import org.jgroups.protocols.raft.StateMachine;
import org.jgroups.raft.RaftHandle;
import org.jgroups.util.AsciiString;
import org.jgroups.util.Bits;
import org.jgroups.util.ByteArrayDataInputStream;
import org.jgroups.util.ByteArrayDataOutputStream;
import org.jgroups.util.Util;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * Distribute counter service based on Raft consensus algorithm.
 */
class CounterService implements StateMachine, RAFT.RoleChange {

    private RaftHandle raft;

    private final Map<String, Long> counters;

    private enum Command {
        CREATE, INCREMENT_AND_GET, DECREMENT_AND_GET, GET, SET
    }

    public CounterService(Channel ch) {
        this.raft = new RaftHandle(ch, this);
        this.counters = new HashMap<>();

        raft.raftId(ch.getName())
            .addRoleListener(this);
    }

    // ===========================================
    //              Raft Status API
    // ===========================================

    public int lastApplied() {
        return raft.lastApplied();
    }

    public int commitIndex() {
        return raft.commitIndex();
    }

    public int logSize() {
        return raft.logSize();
    }

    public void dumpLog() {
        System.out.println("\nindex (term): command\n---------------------");
        raft.logEntries((entry, index) -> {
            StringBuilder log = new StringBuilder()
                    .append(index)
                    .append(" (").append(entry.term()).append("): ");

            if (entry.command() == null ) {
                System.out.println(log.append("<marker record>"));
                return;
            } else if (entry.internal()) {
                System.out.println(log.append("<internal command>"));
                return;
            }

            ByteArrayDataInputStream in = new ByteArrayDataInputStream(
                    entry.command(), entry.offset(), entry.length()
            );
            try {
                Command cmd = Command.values()[in.readByte()];
                String name = Bits.readAsciiString(in).toString();
                switch (cmd) {
                    case CREATE:
                        log.append(cmd)
                            .append("(").append(name).append(", ")
                            .append(Bits.readLong(in))
                            .append(")");
                        break;
                    case GET:
                    case INCREMENT_AND_GET:
                    case DECREMENT_AND_GET:
                        log.append(cmd)
                            .append("(").append(name).append(")");
                        break;
                    default:
                        throw new IllegalArgumentException("Command " + cmd + "is unknown");
                }
                System.out.println(log);
            }
            catch (IOException e) {
                throw new IllegalStateException("Error when dump log", e);
            }
        });
        System.out.println();
    }

    public void snapshot() {
        try {
            raft.snapshot();
        } catch (Exception e) {
            throw new IllegalStateException("Error when snapshot", e);
        }
    }

    public boolean isLeaderExist() {
        return raft.leader() != null;
    }

    // ===========================================
    //              Raft API
    // ===========================================

    @Override
    public void roleChanged(Role role) {
        System.out.println("roleChanged to: " + role);
    }

    @Override
    public byte[] apply(byte[] data, int offset, int length) throws Exception {
        ByteArrayDataInputStream in = new ByteArrayDataInputStream(data, offset, length);
        Command cmd = Command.values()[in.readByte()];
        String name = Bits.readAsciiString(in).toString();
        System.out.println("[" + new SimpleDateFormat("HH:mm:ss.SSS").format(new Date())
                + "] Apply: cmd=[" + cmd + "]");

        long v1, retVal;
        switch (cmd) {
            case CREATE:
                v1 = Bits.readLong(in);
                retVal = create0(name, v1);
                return Util.objectToByteBuffer(retVal);
            case GET:
                retVal = get0(name);
                return Util.objectToByteBuffer(retVal);
            case INCREMENT_AND_GET:
                retVal = add0(name, 1L);
                return Util.objectToByteBuffer(retVal);
            case DECREMENT_AND_GET:
                retVal = add0(name, -1L);
                return Util.objectToByteBuffer(retVal);
            default:
                throw new IllegalArgumentException("Command " + cmd + "is unknown");
        }
    }

    @Override
    public void readContentFrom(DataInput in) throws Exception {
        int size = in.readInt();
        System.out.println("ReadContentFrom: size=[" + size + "]");
        for (int i = 0; i < size; i++) {
            AsciiString name = Bits.readAsciiString(in);
            Long value = Bits.readLong(in);
            counters.put(name.toString(), value);
        }
    }

    @Override
    public void writeContentTo(DataOutput out) throws Exception {
        synchronized (counters) {
            int size = counters.size();
            System.out.println("WriteContentFrom: size=[" + size + "]");
            out.writeInt(size);
            for (Map.Entry<String, Long> entry : counters.entrySet()) {
                AsciiString name = new AsciiString(entry.getKey());
                Long value = entry.getValue();
                Bits.writeAsciiString(name, out);
                Bits.writeLong(value, out);
            }
        }
    }

    // ===========================================
    //              Counter API
    // ===========================================

    public void getOrCreateCounter(String name, long initVal) {
        Object retVal = invoke(Command.CREATE, name, false, initVal);
        counters.put(name, (Long) retVal);
    }

    public long incrementAndGet(String name) {
        return (long) invoke(Command.INCREMENT_AND_GET, name, false);
    }

    public long decrementAndGet(String name) {
        return (long) invoke(Command.DECREMENT_AND_GET, name, false);
    }

    public long get(String name) {
        return (long) invoke(Command.GET, name, false);
    }

    private Object invoke(Command cmd, String name, boolean ignoreRetVal, long... values) {
        ByteArrayDataOutputStream out = new ByteArrayDataOutputStream(256);
        try {
            out.writeByte(cmd.ordinal());
            Bits.writeAsciiString(new AsciiString(name), out);
            for (long val : values) {
                Bits.writeLong(val, out);
            }

            byte[] rsp = raft.set(out.buffer(), 0, out.position());
            return ignoreRetVal ? null : Util.objectFromByteBuffer(rsp);
        }
        catch (IOException ex) {
            throw new RuntimeException("Serialization failure (cmd="
                    + cmd + ", name=" + name + ")", ex);
        }
        catch (Exception ex) {
            throw new RuntimeException("Raft set failure (cmd="
                    + cmd + ", name=" + name + ")", ex);
        }
    }

    // ===========================================
    //              Counter Native API
    // ===========================================

    public synchronized Long create0(String name, long initVal) {
        counters.putIfAbsent(name, initVal);
        return counters.get(name);
    }

    public synchronized Long get0(String name) {
        return counters.getOrDefault(name, 0L);
    }

    public synchronized Long add0(String name, long delta) {
        Long oldVal = counters.getOrDefault(name, 0L);
        return counters.put(name, oldVal + delta);
    }
}


```

###3.3 运行测试

我们分别以A、B、C为参数，启动三个JGroupsRaftTest服务。这样会自动在C:\Users\cdai\AppData\Local\Temp下生成A.log、B.log、C.log三个日志文件夹。

```

cdai@vm /cygdrive/c/Users/cdai/AppData/Local/Temp
$ tree A.log/ B.log/ C.log/
A.log/
|-- 000005.sst
|-- 000006.log
|-- CURRENT
|-- LOCK
|-- LOG
|-- LOG.old
`-- MANIFEST-000004
B.log/
|-- 000003.log
|-- CURRENT
|-- LOCK
|-- LOG
`-- MANIFEST-000002
C.log/
|-- 000003.log
|-- CURRENT
|-- LOCK
|-- LOG
`-- MANIFEST-000002

0 directories, 17 files

```

###3.3.1 分布式一致性

首先A创建计数器，B“加一”，C“减一”。可以看到尽管我们是分别在A、B、C上执行这三个操作，但三个结点都先后（leader提交日志后通知follower）通过apply()方法收到消息，并在本地的计数器Map上同步执行操作，保证了数据的一致性。最后停掉A服务，可以看到B通过roleChanged()得到消息，提升为新的Leader，并与C一同继续提供服务。

A的控制台输出：

```

-------------------------------------------------------------------
GMS: address=A, cluster=ctr-cluster, physical address=2001:0:9d38:6abd:cbb:1f78:3f57:50f6:50100
-------------------------------------------------------------------

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=0, commit-index=0, log size=0b: 
roleChanged to: Candidate
roleChanged to: Leader
0
[14:16:00.744] Apply: cmd=[CREATE]

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=1, commit-index=1, log size=1b: 
[14:16:07.002] Apply: cmd=[INCREMENT_AND_GET]
[14:16:14.264] Apply: cmd=[DECREMENT_AND_GET]
3

index (term): command
---------------------
1 (29): CREATE(counter, 1)
2 (29): INCREMENT_AND_GET(counter)
3 (29): DECREMENT_AND_GET(counter)

```

B的控制台输出：

```
-------------------------------------------------------------------
GMS: address=B, cluster=ctr-cluster, physical address=2001:0:9d38:6abd:cbb:1f78:3f57:50f6:50101
-------------------------------------------------------------------

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=0, commit-index=0, log size=0b: 
[14:16:01.300] Apply: cmd=[CREATE]
1
counter: 2

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=2, commit-index=1, log size=2b: 
[14:16:07.299] Apply: cmd=[INCREMENT_AND_GET]
[14:16:14.304] Apply: cmd=[DECREMENT_AND_GET]
roleChanged to: Candidate
roleChanged to: Leader

```

C的控制台输出：

```

-------------------------------------------------------------------
GMS: address=C, cluster=ctr-cluster, physical address=2001:0:9d38:6abd:cbb:1f78:3f57:50f6:55800
-------------------------------------------------------------------

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=0, commit-index=0, log size=0b: 
[14:16:01.300] Apply: cmd=[CREATE]
[14:16:07.299] Apply: cmd=[INCREMENT_AND_GET]
2
counter: 3

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=3, commit-index=2, log size=3b: 
[14:16:14.304] Apply: cmd=[DECREMENT_AND_GET]

```

###3.3.2 服务恢复

在只有B和C的集群中，我们执行了一次“加一”。当我们重新启动A服务时，它会自动执行这条日志，保持与B和C的一致。从日志的index能够看出，69是一个Term，也就是A为Leader时的“任期”，而70也就是B为Leader时。

A的控制台输出：

```
-------------------------------------------------------------------
GMS: address=A, cluster=ctr-cluster, physical address=2001:0:9d38:6abd:cbb:1f78:3f57:50f6:53237
-------------------------------------------------------------------

[0] Create [1] Increment [2] Decrement [3] Dump log [4] Snapshot [x] Exit
first-applied=0, last-applied=3, commit-index=3, log size=3b: 
[14:18:45.275] Apply: cmd=[INCREMENT_AND_GET]
[14:18:45.277] Apply: cmd=[GET]
3

index (term): command
---------------------
1 (69): CREATE(counter, 1)
2 (69): INCREMENT_AND_GET(counter)
3 (69): DECREMENT_AND_GET(counter)
4 (70): INCREMENT_AND_GET(counter)
5 (70): GET(counter)

```