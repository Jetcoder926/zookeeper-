# ZOOKEEPER学习笔记

**Zk是一个提供分布式协同服务的应用.例如集群服务器的数据的一致性、参数配置的一致性**


![image](https://colobu.com/images/logos/zookeeper.png)

## 应用场景
zk最适于读多写少且轻量级数据（默认设置下单个dataNode限制为1MB大小,可通过Java的环境变量来设置）的应用场景

## 运行原理:
zk通过创建znode节点,当客户端连接节点时,节点会分配会话ID并向客户端发确认函. 当客户端请求数据变化时. 会产生一个watcher监听事件(监听事件是用来处理当数据发生变化时处理回调方法).

<br/><br/>
> ## 问答环节

### 1.客户端连接过程中.如果节点挂了怎么办
##### &emsp;&emsp;&emsp;Zk的每个znode都会维护一个stat，由版本号、 操作控制列表(ACL)，时间戳和数据长度组成。而节点会在一个配置好的周期时间发送确认给客户端.如果客户端接收超时.则会自动连到其他节点.操作也会立刻进行

### 2.客户端写入数据到znode的过程时怎样的
##### &emsp;&emsp;&emsp;Zk是主写从读的设计，结构是一个master/多个follower，而其中涉及到数据复制的机制，由leader进行写入同时广播至follower，过半成功响应了(保证其数据一定能复制及q1中的情况).则返回写入成功.而写入具有原子性，只有失败与成功2种结果

### 3.一个客户端在写入的时候.另一个客户端在读取.数据是最新的吗
##### &emsp;&emsp;&emsp; zk不保证读一致性，默认是弱一致性，如果要保证读到的数据是最新的，读取之前要使用sync方法(强一致性)

### 4.leader对于一个事务在本地提交了，但是还没广播就down机了，那么从其余follow中选出的leader如何保证这个事务也被提交？
##### &emsp;&emsp;&emsp; 旧leader commit完就挂掉了，因为写入的follower肯定超过一半，新leader具有最大zxid，因此新leader就拥有commit的proposal，这时候只要提交该proposal并进行同步即可。如果旧leader只同步了一个follower就挂掉，该follower就是新leader，新leader同步该proposal然后同步即可。如果旧leader还没来得及同步就挂掉，该proposal在新集群中也不会存在，也不会成功，因此当旧leader恢复时就会被rollback
