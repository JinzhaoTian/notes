

## 架构


1. 直接采用互联网架构：nginx做网关，上一个redis集群做缓存，后面mysql分库分表，游戏内部逻辑给客户端一丢，服务器做的逻辑相对于其他游戏类型会比较少。
2. MMO 游戏服务器架构设计。


## 开源项目

1. [Noah](https://github.com/ketoo/NoahGameFrame) 




## 算法

### 寻路算法

BFS、Dijkstra和A Star是图上的三种典型路径规划算法。它们都可用于图搜索，不同之处在于队列和启发式函数两个参数。

1. **BFS**：先进先出队列实现BFS
![](imgs/Pasted%20image%2020230704135632.png)

2. **Dijkstra**：使用优先级队列，Dijkstra算法可以非常方便的找出从地图上某个起始区块到其他所有可达区块的最短路径。
![](imgs/Pasted%20image%2020230704135626.png)
```Properties
function Dijkstra(Graph, source):
    for each vertex v in Graph.Vertices:
        dist[v] ← INFINITY
        prev[v] ← UNDEFINED
        add v to Q
    dist[source] ← 0

    while Q is not empty:
        u ← vertex in Q with min dist[u]
        remove u from Q

        for each neighbor v of u still in Q:
            alt ← dist[u] + Graph.Edges(u, v)
            if alt < dist[v]:
                dist[v] ← alt
                prev[v] ← u
    
    return dist[], prev[]
```

  

3. **A\*寻路算法**：利用到启发式函数，基于格子（Grid）的寻路算法，也就是说会把我们的地图看作是由 w*h 个格子组成的，因此寻得的路径也就是由一连串相邻的格子所组成的路径。 通过评估来找到合适路径的算法我们称之为**启发式算法**，即**优先搜索最有可能产生最佳路径的格子**。
![](imgs/Pasted%20image%2020230704135700.png)

```
// 伪代码
function reconstruct_path(cameFrom, current)    
    total_path := {current}
    while current in cameFrom.Keys:
        current := cameFrom[current]
        total_path.prepend(current)
        return total_path
        
// A* finds a path from start to goal.
// h is the heuristic function. h(n) estimates the cost to reach goal from node n.
function A_Star(start, goal, h)    
    // The set of discovered nodes that may need to be (re-)expanded.    
    // Initially, only the start node is known.    
    // This is usually implemented as a min-heap or priority queue rather than a hash-set.    
    openSet := {start}    
    
    // For node n, cameFrom[n] is the node immediately preceding it on the cheapest path from the start    
    // to n currently known.    
    cameFrom := an empty map    
    
    // For node n, gScore[n] is the cost of the cheapest path from start to n currently known.    
    gScore := map with default value of Infinity    
    gScore[start] := 0    
    
    // For node n, fScore[n] := gScore[n] + h(n). fScore[n] represents our current best guess as to    
    // how cheap a path could be from start to finish if it goes through n.    
    fScore := map with default value of Infinity    
    fScore[start] := h(start)    
    
    while openSet is not empty        
        // This operation can occur in O(Log(N)) time if openSet is a min-heap or a priority queue        
        current := the node in openSet having the lowest fScore[] value        
        if current = goal            
            return reconstruct_path(cameFrom, current)        
            
        openSet.Remove(current)        
        for each neighbor of current            
            // d(current,neighbor) is the weight of the edge from current to neighbor            
            // tentative_gScore is the distance from start to the neighbor through current            
            tentative_gScore := gScore[current] + d(current, neighbor)            
            if tentative_gScore < gScore[neighbor]                
                // This path to neighbor is better than any previous one. Record it!                
                cameFrom[neighbor] := current                
                gScore[neighbor] := tentative_gScore                
                fScore[neighbor] := tentative_gScore + h(neighbor)                
                
                if neighbor not in openSet                    
                    openSet.add(neighbor)    
                    
    // Open set is empty but goal was never reached    
    return failure
```

## 通信协议

  
JSON 因为其轻量和易阅读，对开发者友好逐步替代了 XML ，所以现在大多数的前后端交互都是使用的 JSON 。但是由于容器化、K8s 的崛起，[ProtoBuf](../../../Language/Data%20Format/ProtoBuf.md) 作为一种跨平台的序列化结构化数据的协议慢慢的开始了崭露头角。


## 高性能网络模式

1. [IO 多路复用](../../../Computer%20Network/IO%20多路复用.md)
2. [IO 设计模式](../../../Computer%20Network/IO%20设计模式.md)


## 笔试题

1. 堆和栈的区别
    
2. 线程、进程和协程各自的优劣和区别 **进程**是操作系统资源分配的基本单位，**线程**是任务调度和执行的基本单位。一个进程中的所有线程共享地址空间，因此全局变量，指针，引用可以在线程之间传递。 **协程**就是一段可以挂起（suspend）和恢复（resume）的程序，一般而言，就是一个支持挂起和恢复的函数。
    
3. 寻路算法
    
4. 1亿个int数字，从中选择出现频率最高的100个 考虑采用hash_map/搜索二叉树/红黑树等来进行统计次数。然后就是取出前N个出现次数最多的数据了，可以用堆机制完成。
    
5. 判断用户名不重复 需要分布式锁。问题被拆分成了两个，如何实现分布式锁 + 如何尽量减少锁的粒度。实现分布式锁最简单的方式就是用Redis。
    
6. Redis的Cluster集群使用的分片算法 虚拟哈希槽
    
7. 基于Redis设计分布式锁
    
8. MySQL中SQL的查询过程
    
9. 场景：存在一个MySQL表，有A、B、C、D、E五个字段。线上发现，C、D、E字段大多数场景下不需要，请设计索引，并说明原因。
    
10. 请介绍下protobuf和json相比的优缺点
    
11. 请介绍protobuf的index兼容是如何实现
    
12. 请介绍下游戏中常见的ping值显示如何实现 客户端记录当前时间并发送数据包给服务器端(tcp长连接可用心跳)，服务器收到后返回指定包。客户端收到返回包的时间减去之前记录的时间就是ping。
    
13. 场景：企业微信中，有一个功能是“已读”，请介绍思路实现这个功能，需要满足1000人以上的大群 如果是**私聊**：消息的阅读状态比较容易实现，在性能和存储上也不存在问题； 如果是**群聊**：考虑到存储和处理性能，特别当处于一个云环境时，如何高效地处理群聊的已读未读状态是一个非常值得探讨的话题。“高效”包含3个方面：存储空间、处理速度、传输字节数。 可以对于每个消息维护一个bitmap，然后保存userid到自增mapid的映射
    
14. 场景：假设你在吃鸡世界里，有1w个玩家在一个场景中，每秒钟会产生1w发子弹，每个子弹有射程上限，请介绍一种思路快判断子弹是否击中玩家。
    
15. 场景：滴滴打车如何给我分配最近的司机 多对多问题，二部图匹配，
    
16. 场景：外卖软件里，请设计一个算法，快速给到用户筛选出附近的商家 GeoHash是一种地址编码方法。他能够把二维的空间经纬度数据编码成一个字符串，首先将经纬度变成二进制，第2步，就是将经纬度合并，第3步，按照Base32进行编码。GeoHash表示的并不是一个点，而是一个矩形区域。
    ![](imgs/Pasted%20image%2020230704135729.png)
17. 场景：吃鸡游戏里如何屏蔽透视外挂 FPS游戏品类存在天然“缺陷”。为保障射击类游戏的流畅度体验，许多玩法的计算逻辑需要放在客户端本地进行，无法采取服务计算校验的方式。这种方式和卡牌等无需注重客户端实时计算体验的游戏品类有很大不同。大量计算逻辑放在客户端，也就为外挂作弊埋下了伏笔，这正是射击类游戏作弊门槛低、容易被外挂侵袭的原因所在。 常规的修改数据类外挂，从技术手段入手还是比较容易识别，但跨进程类的外挂就更难防御了。它是独立于游戏的第三方进程，以高权限方式跨进程读取游戏关键逻辑数据，通过进程外绘制实现透视等外挂功能。跨进程透视外挂只读取游戏关键数据，游戏逻辑层执行没有任何异常，游戏无法感知到外挂进程相关的详细信息。所以一般的反作弊手段是无法处理此类外挂的。 修改shader类作弊方式，该作弊方式是通过修改shader数据可以影响GPU的渲染过程，可以实现人物渲染类透视、人物上色、除草除树等功能。 除了手机端的作弊之外，在PC通过模拟器运行手游的同时使用外挂。 实现外挂功能，最常见的就是使用外挂插件，修改游戏数据、内存等方式。
    

- 包id自增校验，可以消灭WPE（网络封包编辑器）
    
- 包校验码可以消灭或者拦截篡改的包
    
- 图形识别码，可以踢掉 99% 非人的操作
    

  

18. 场景：王者荣耀有几亿玩家，如何给每个玩家生成唯一id 常见的分布式全局唯一ID生成方式包括使用数据库自增，使用Redis的原子操作INCR和INCRBY，使用UUID，SnowFlake算法等等。前面两种方式均需要产生一次异步调用，在MMO中，海量玩家会集中在一个场景中进行PK，做任务，打怪等，场景内业务逻辑复杂，为了降低编码复杂度，减少BUG几率，**通常会选择使用本地算法来生成全局唯一ID**。 UUID方式生成的ID比较长，通常需用字符串表示，作为内存数据主键或者数据库主键它的查找效率比不上直接使用整数类型生成的ID做主键。同时它对业务来说是一串无规则的字符串，不能根据相关业务规则进行调整。 **SnowFlake算法**是twitter开源的分布式ID生成算法，它是一个本地生成算法，它可以生成一个64位的整数。
    
19. 请介绍下网络Reactor模型和Proactor模型的区别
    
20. 游戏服务器序列化/反序列化用什么样的技术？ 序列化/反序列化的概念：序列化是指将对象实例的状态存储到存储媒体的过程；反序列化是指将存储在存储媒体中的对象状态转换成对象的过程。 目前服务器序列化与反序列化主要分成两种模式二进制模式与文本模式，文本模式的序列化与反序列化主要有json与xml, 二进制模式的序列化与反序列化主要有自定义的协议和google的protobuf协议。