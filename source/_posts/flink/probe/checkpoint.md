---
    title: checkpoint
    creator: cjq
    create_time: 2026/03/03
    tags:
      - flink
    categories:
      - [flink, probe]

---



[toc]



# 保存算法

## checkpoint保存算法有几种

Flink的Checkpoint机制是其容错和状态一致性的核心。根据触发机制、数据保存方式和处理逻辑的不同，Checkpoint的实现方式可以从几个不同的维度进行分类。

### 🔀 按屏障对齐方式分类

| 特性             | 对齐检查点 (Aligned Checkpoint)                              | 非对齐检查点 (Unaligned Checkpoint)                          |
| :--------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **核心机制**     | 基于Chandy-Lamport分布式快照算法变体--，通过Barrier对齐保证快照一致性。 | 在Flink 1.11引入，1.13达到生产可用[-2](https://flink-learning.org.cn/article/detail/643e3d7864edaee92e117b7b7aaa26b7?name=author&page=xiangguanwenzhang)，优化反压场景下的Checkpoint性能[-33](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=188745770&navigatingVersions=true)。 |
| **触发时机**     | 当算子**最后一个**输入流的Barrier到达时触发-[-10](https://developer.aliyun.com/ask/651536)。 | 当算子**第一个**输入流的Barrier到达时就触发-[-10](https://developer.aliyun.com/ask/651536)。 |
| **阻塞情况**     | Barrier对齐期间会**阻塞**后续数据处理-[-10](https://developer.aliyun.com/ask/651536)。 | **无需阻塞**数据处理，Barrier会越过正在处理的数据[-10](https://developer.aliyun.com/ask/651536)[-33](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=188745770&navigatingVersions=true)。 |
| **状态大小**     | 仅保存算子状态，状态大小相对较小[-10](https://developer.aliyun.com/ask/651536)。 | 额外保存Barrier之后“正在传输中”的数据，状态大小可能膨胀，每个Task可能达到几GB[-10](https://developer.aliyun.com/ask/651536)。 |
| **适用场景**     | 状态较小、反压不严重的常规场景。                             | **反压严重**、需要保证Checkpoint快速完成的场景-[-10](https://developer.aliyun.com/ask/651536)。 |
| **Exactly-Once** | 严格保证。                                                   | 同样保证Exactly-Once语义[-10](https://developer.aliyun.com/ask/651536)[-11](https://cloud.tencent.cn/developer/article/1972196?from=15425)。 |

> 从Flink 1.13开始，官方引入了超时对齐机制，即“对齐超时后自动切换为非对齐”的混合模式，以平衡两种方式的优缺点-。

### 📊 按状态备份方式分类

| 特性               | 全量检查点 (Full Checkpoint)                                 | 增量检查点 (Incremental Checkpoint)                          |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **核心机制**       | 每次Checkpoint都**完整备份**所有状态数据[-6](https://www.dtstack.com/bbs/article/124234)。 | 仅备份自上次Checkpoint以来的**状态变化部分**[-5](https://blog.csdn.net/JiShuiSanQianLi/article/details/127531616)[-21](https://flink-learning.org.cn/article/detail/6636e468bc961f789f8d0ba8ffd51a95?name=article&tab=suoyou&page=37)。 |
| **存储开销**       | 较大，每次备份完整状态。                                     | **显著降低**存储开销，尤其适合大状态场景[-6](https://www.dtstack.com/bbs/article/124234)[-21](https://flink-learning.org.cn/article/detail/6636e468bc961f789f8d0ba8ffd51a95?name=article&tab=suoyou&page=37)。 |
| **Checkpoint时间** | 较长，与状态大小成正比。                                     | **大幅缩短**Checkpoint时间（TB级作业可从3分钟降至30秒）[-21](https://flink-learning.org.cn/article/detail/6636e468bc961f789f8d0ba8ffd51a95?name=article&tab=suoyou&page=37)。 |
| **恢复时间**       | 较快，直接加载完整快照。                                     | 相对较长，需要从基础快照+多个增量文件恢复-。                 |
| **适用场景**       | 状态较小、对恢复时间要求高的场景。                           | **状态巨大**（GB/TB级别）、希望减少Checkpoint开销的场景[-6](https://www.dtstack.com/bbs/article/124234)。 |
| **支持状态后端**   | 所有状态后端。                                               | 主要支持`RocksDBStateBackend`[-21](https://flink-learning.org.cn/article/detail/6636e468bc961f789f8d0ba8ffd51a95?name=article&tab=suoyou&page=37)。 |

### ⚡ 按执行阶段分类

| 特性               | 同步阶段 (Synchronous Phase)                                 | 异步阶段 (Asynchronous Phase)                                |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **执行内容**       | 调用`snapshotState()`方法，进行状态快照的**准备或浅拷贝**[-19](https://flink-learning.org.cn/article/detail/3ef2695bda7fa6b11cca814381cb2c16?name=article&page=16)。 | 将同步阶段准备好的状态数据**持久化**到远程存储[-18](https://flink-learning.org.cn/article/detail/abdd791ed070da92981825da780b7d3d?name=article&tab=suoyou&page=17)。 |
| **对数据处理影响** | **会阻塞**数据处理，是Checkpoint的停顿时间。                 | **不阻塞**数据处理，后台线程执行上传[-2](https://flink-learning.org.cn/article/detail/643e3d7864edaee92e117b7b7aaa26b7?name=author&page=xiangguanwenzhang)。 |
| **优化方向**       | 优化同步阶段的逻辑，减少状态复制开销。                       | 通过增量Checkpoint等机制减少上传数据量，或通过通用增量Checkpoint架构解耦。 |

### 🚀 高级优化：通用增量检查点 (Generic Log-Based Incremental Checkpoint)

> 在Flink 1.15引入，1.16达到生产可用[-1](https://www.alibabacloud.com/en/developer/a/flink/table-general-incremental-checkpoint?_p_lc=1)[-18](https://flink-learning.org.cn/article/detail/abdd791ed070da92981825da780b7d3d?name=article&tab=suoyou&page=17)

这是一种更先进的增量Checkpoint架构，主要优化包括：

- **核心机制**：引入State Changelog机制，将状态更新的变化日志**持续**写入远程存储（如HDFS/S3），而Checkpoint仅需Flush这些日志[-1](https://www.alibabacloud.com/en/developer/a/flink/table-general-incremental-checkpoint?_p_lc=1)[-18](https://flink-learning.org.cn/article/detail/abdd791ed070da92981825da780b7d3d?name=article&tab=suoyou&page=17)。
- **异步优化**：将状态持久化与Checkpoint过程**完全解耦**，避免大文件上传的不确定性[-18](https://flink-learning.org.cn/article/detail/abdd791ed070da92981825da780b7d3d?name=article&tab=suoyou&page=17)。
- **主要优势**：Checkpoint完成时间更**稳定、快速**，端到端延迟更低[-1](https://www.alibabacloud.com/en/developer/a/flink/table-general-incremental-checkpoint?_p_lc=1)[-18](https://flink-learning.org.cn/article/detail/abdd791ed070da92981825da780b7d3d?name=article&tab=suoyou&page=17)。

------

### 💎 总结

Flink的Checkpoint算法可以根据不同维度进行分类：

- **按屏障对齐方式**：分为**对齐**和**非对齐**两种，主要权衡反压场景下的Checkpoint完成速度与额外的存储开销。
- **按状态备份方式**：分为**全量**和**增量**两种，增量方式极大地优化了大状态作业的Checkpoint性能。
- **按执行阶段**：分为**同步**和**异步**两个阶段，将耗时的持久化操作异步化，减少对数据处理路径的影响。
- **高级优化**：**通用增量Checkpoint**（1.15+）是面向未来的架构，通过解耦Checkpoint与状态存储，实现了更稳定高效的Checkpoint。

这些算法通常**组合使用**。例如，一个生产作业可能配置为：使用**RocksDB状态后端**、开启**增量Checkpoint**、并设置**对齐超时自动切换为非对齐**。



## 对齐算法中 Barrier对齐期间会阻塞后续数据处理 是什么意思

简单来说，**“Barrier对齐期间会阻塞后续数据处理”** 指的是Flink为了保证数据一致性，在Exactly-Once模式下执行检查点时，会采取的一种“停下来等”的策略。这个策略确实会导致数据处理出现短暂的“暂停”，以换取状态快照的绝对精准。

### ⏸️ 阻塞场景：为什么必须“等”？

要理解阻塞，需要先了解Flink的Barrier对齐机制。当JobManager触发一次Checkpoint时，Source算子会向数据流中注入带有ID的Barrier（屏障）[-11](https://blog.csdn.net/lifallen/article/details/148476015)。对于**只有一个输入流**的算子，Barrier的处理相对简单，它会在收到Barrier时直接触发快照[-23](https://blog.csdn.net/zznanyou/article/details/121347608#comments_25443477)。

阻塞主要发生在**有多个输入流的算子（如`CoProcessFunction`, `Join`等）**上[-23](https://blog.csdn.net/zznanyou/article/details/121347608#comments_25443477)。因为不同的输入流数据到达速度可能不同，Barrier到达的时间也会有先后-。

为了保证快照能精准地反映Barrier到达前那一刻所有输入流的状态，算子必须执行“对齐”操作：

1. **先到的先等**：当算子从某个输入流（比如`输入流A`）收到了一个Barrier（编号为`n`）时，它立即停止处理**该输入流**Barrier之后的任何新数据[-23](https://blog.csdn.net/zznanyou/article/details/121347608#comments_25443477)。
2. **继续处理其他**：算子会继续处理其他尚未收到Barrier的输入流（比如`输入流B`）中的数据[-11](https://blog.csdn.net/lifallen/article/details/148476015)。
3. **对齐完成**：直到`输入流B`的Barrier（编号也是`n`）也到达后，所有输入流的Barrier才算“对齐”[-11](https://blog.csdn.net/lifallen/article/details/148476015)。这时，算子才会触发自己的状态快照，并向下游广播Barrier[-23](https://blog.csdn.net/zznanyou/article/details/121347608#comments_25443477)。

这个过程中，从第一个Barrier到达，到最后一个Barrier到达的这段时间，就是“对齐时间”（Alignment Duration）[-2](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/ops/state/large_state_tuning/index.html)-。在此期间，`输入流A`的处理被阻塞，`输入流A`后续的数据会被暂存在一个缓冲区里，等待对齐完成后才被继续处理[-23](https://blog.csdn.net/zznanyou/article/details/121347608#comments_25443477)[-25](https://www.modb.pro/db/81196)。如果这个过程非常耗时，就说明作业可能正经历反压[-2](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/ops/state/large_state_tuning/index.html)。

### 🆚 两种语义：阻塞与不阻塞的区别

这种“阻塞”行为，是实现 **Exactly-Once** 的关键，它能确保所有输入流的状态在快照时是完全一致的。

如果为了追求低延迟而选择“不等待”，即一旦接收到任何一个Barrier就立即触发快照并继续处理所有数据，这就变成了 **At-Least-Once** 语义。这样虽然减少了阻塞，但可能在恢复时造成数据重复处理[-7](https://juejin.cn/post/7126119375530098724?from=search-suggest)。

### 💡 阻塞的影响：反压与性能

“阻塞”虽然保证了准确性，但也可能带来一些副作用：

- **加剧反压**：在数据倾斜或下游处理能力不足时，对齐过程中的等待会进一步减慢数据处理速度，导致Barrier流动缓慢，Checkpoint耗时变长[-11](https://blog.csdn.net/lifallen/article/details/148476015)。
- **增加内存开销**：用于暂存阻塞数据的输入缓冲区会消耗内存。如果对齐时间过长，缓冲区积压的数据会增多，可能导致内存压力过大[-23](https://blog.csdn.net/zznanyou/article/details/121347608#comments_25443477)。

### 🔧 解决方案：非对齐检查点 (Unaligned Checkpoint)

为了解决“对齐阻塞”在反压场景下带来的性能问题，Flink从1.11版本开始引入了 **非对齐检查点 (Unaligned Checkpoint)** [-7](https://juejin.cn/post/7126119375530098724?from=search-suggest)。

它的核心思路是 **“不停下来等，而是把等待时正在传输的数据也保存起来”**。当算子的某个输入流收到Barrier后，它不再阻塞该通道，而是立即开始做快照，并将所有输入和输出缓冲区中“正在飞行”的数据作为快照的一部分一起持久化[-11](https://blog.csdn.net/lifallen/article/details/148476015)。<font color=#FF0000 >这相当于把对齐的压力从运行时转移到了恢复时</font>，有效避免了因对齐阻塞而加剧反压的问题[-11](https://blog.csdn.net/lifallen/article/details/148476015)。

### 💎 总结

“Barrier对齐阻塞”是Flink为换取Exactly-Once语义而设计的机制，它通过在快照制作时暂停部分数据处理来确保状态的一致性。这个机制在反压时可能成为瓶颈，而非对齐检查点正是为了解决这一问题而生的。



## 书中exactly once语义一节中看到一句话，其中一个步骤是算子任务对状态做快照，书上说，当分界线传递到了内部的window()算子，它同样会对自己的状态进行快照保存，并写入远程的持久化存储中。状态后端不是有两种吗，其中一种是写入内存的，为什么这里直接说写入持久化存储了？

这个观察很敏锐。书中的描述其实是从实现原理的角度出发的。对于Flink来说，只要开启了Checkpoint，即使使用看起来只存在内存中的`MemoryStateBackend`，状态也会以**文件形式被持久化到远程存储**。这其实是Flink实现Exactly-Once语义的基石。

`MemoryStateBackend`这个名称确实容易引起误解，它指的主要是运行时状态在内存中，但Checkpoint数据依然需要可靠的持久化。

------

### 🧐 一个名字引发的误解：`MemoryStateBackend`

根据Flink官方文档，`MemoryStateBackend`的核心行为是：

- **运行时状态**：保存在TaskManager的JVM堆内存中[-1](https://nightlies.apache.org/flink/flink-docs-release-1.19/api/java/org/apache/flink/runtime/state/memory/MemoryStateBackend.html#createKeyedStateBackend-org.apache.flink.runtime.state.StateBackend.KeyedStateBackendParameters-)。
- **Checkpoint时**：状态数据被发送到JobManager，并**为了高可用性（HA）被持久化到文件系统中**[-1](https://nightlies.apache.org/flink/flink-docs-release-1.19/api/java/org/apache/flink/runtime/state/memory/MemoryStateBackend.html#createKeyedStateBackend-org.apache.flink.runtime.state.StateBackend.KeyedStateBackendParameters-)。它本质上是一个可以脱离文件系统工作的“基于文件系统的后端”[-1](https://nightlies.apache.org/flink/flink-docs-release-1.19/api/java/org/apache/flink/runtime/state/memory/MemoryStateBackend.html#createKeyedStateBackend-org.apache.flink.runtime.state.StateBackend.KeyedStateBackendParameters-)。

这种设计在Flink 1.12的官方文档里说得更直接：`MemoryStateBackend`在Checkpoint时，**“将快照信息作为CheckPoint应答消息的一部分发送给JobManager(master)，同时JobManager也将快照信息存储在堆内存中”**[-5](https://nightlies.apache.org/flink/flink-docs-release-1.12/zh/ops/state/state_backends.html)。这里并没有提及直接持久化到远程存储，所以书里的说法，很可能是基于更高版本中对其HA机制的描述，或是从“实现Exactly-Once语义”这个更高阶的视角出发，将Checkpoint视为**最终必须写入持久化存储**的行为。

### 💡 概念重塑：状态后端与检查点存储的“解耦”

Flink 1.15及之后版本的重要演进，可以帮助我们更好地理解这种设计：**Flink将状态后端（State Backend）和检查点存储（Checkpoint Storage）这两个概念彻底分开了**[-3](https://cwiki.apache.org/confluence/pages/diffpages.action?originalId=158878190&pageId=158878672)。这意味着，你完全可以组合使用不同的后端和存储方式：

- `HashMapStateBackend` (等价于`MemoryStateBackend`) 处理运行时状态。
- `JobManagerCheckpointStorage` 负责将Checkpoint数据存储在JobManager内存中，适用于本地测试[-3](https://cwiki.apache.org/confluence/pages/diffpages.action?originalId=158878190&pageId=158878672)。

但为了实现生产级的容错（Exactly-Once语义），通常会将`HashMapStateBackend`与`FileSystemCheckpointStorage`搭配使用，这样状态快照就会被可靠地写入HDFS或S3等分布式文件系统[-3](https://cwiki.apache.org/confluence/pages/diffpages.action?originalId=158878190&pageId=158878672)。

------

### ⚙️ 三种官方状态后端如何工作

下图清晰地展示了Flink官方提供的三种状态后端，在工作状态存储和Checkpoint持久化上的分工：

| 状态后端                              | 工作状态存储 (TaskManager 本地)                              | Checkpoint 持久化存储 (远程)                                 |
| :------------------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `MemoryStateBackend` (旧版，已不推荐) | TaskManager JVM 堆内存[-2](https://www.dtstack.com/bbs/article/456253) | JobManager 内存 (HA模式下可持久化到文件系统)[-1](https://nightlies.apache.org/flink/flink-docs-release-1.19/api/java/org/apache/flink/runtime/state/memory/MemoryStateBackend.html#createKeyedStateBackend-org.apache.flink.runtime.state.StateBackend.KeyedStateBackendParameters-) |
| `FsStateBackend` (旧版，已不推荐)     | TaskManager 内存[-2](https://www.dtstack.com/bbs/article/456253) | 远程文件系统 (如 HDFS, S3)[-2](https://www.dtstack.com/bbs/article/456253) |
| `RocksDBStateBackend` (推荐)          | 本地 RocksDB 实例 (内存+磁盘)[-2](https://www.dtstack.com/bbs/article/456253) | 远程文件系统[-2](https://www.dtstack.com/bbs/article/456253) |

### 💎 总结

所以，书上的那句话并没有错。它强调的是Exactly-Once语义实现的关键一环——**将全局一致的状态快照进行持久化存储**，而具体存到哪里，则由所选的状态后端和检查点存储配置共同决定。`MemoryStateBackend`虽然运行时状态在内存中，但为了实现生产级的容错，它的Checkpoint数据最终也需要被写入可靠的存储中。



## 也就是说，当开启了checkpoint，状态可能会存两份，一份在内存，一份要持久化？

是的，你的理解非常准确。开启 Checkpoint 后，Flink 的状态确实会存在“两份”：一份是用于处理实时数据的**工作状态（Working State）**，另一份是用于故障恢复的**快照状态（Snapshot State）**。

这“两份”数据虽然内容上有关联，但它们的**存储位置、生命周期和作用**是完全不同的。

### 两份状态的详细对比

| 特性         | 工作状态 (Working State)                                     | 快照状态 (Snapshot State / Checkpoint)       |
| :----------- | :----------------------------------------------------------- | :------------------------------------------- |
| **作用**     | 用于处理实时数据，例如存储窗口中的元素、键控状态的值。       | 用于故障恢复，是一个**全局一致的状态副本**。 |
| **存储位置** | 存储在 **TaskManager 本地**。根据状态后端不同，可能在内存或RocksDB。 | 存储在 **远程持久化存储**（如 HDFS, S3）。   |
| **生命周期** | 随作业持续运行，不断被读写和更新。                           | 周期性生成，且是**只读**的，生成后不再改变。 |
| **数量**     | 一份（每个算子实例一个）。                                   | 可能有多份（由 `num-retained` 控制）。       |
| **是否必须** | 是，数据处理的基础。                                         | 是，容错机制的基础。                         |

### 一个类比：文档编辑的“自动保存”

你可以把它们想象成正在编辑一份文档：

- **工作状态** 就是你屏幕上正在编辑的文档内容，你所有的修改都在这里实时发生。它就相当于保存在你电脑内存/缓存里的“工作状态”。
- **快照状态** 就是你每隔一段时间点击“保存”后，存到硬盘上的那个文件。那个文件是过去某个时间点的**静态副本**，主要用于系统崩溃后恢复。

**而开启 Checkpoint 后的过程就像这样：**

1. 你的文档（工作状态）一直在内存中，你可以随时编辑（处理数据）。
2. 每隔几分钟，系统自动帮你把当前文档内容完整地存一次盘（生成快照）。
3. 你的电脑内存里有一份正在编辑的文档（工作状态），硬盘上则可能保留着最近几次存盘的副本（快照状态）。

### 两点补充说明

1. **“两份”不等于“双倍”开销**
   - 虽然数据是两份，但生成快照的过程（特别是异步和增量快照）已经高度优化，通常不会让数据处理的延迟翻倍。
   - 增量快照每次只保存**自上次快照以来的变化部分**，而不是全部数据，这大大减少了快照的数据量和网络传输。
2. **不是所有状态后端都“在内存”**
   - 如果使用 `RocksDBStateBackend`，工作状态本身主要存储在本地磁盘（RocksDB），而不仅仅是内存。这时，“两份”的含义就变成了：**一份在本地磁盘（工作状态）**，另一份在**远程存储（快照）**。

### 总结

所以，你的理解完全正确：Checkpoint 机制本质上就是用**远程存储的静态快照**，为**本地运行的动态工作状态**提供了一层可靠的容错保障。这是一套非常经典的“动态数据 + 静态备份”的分布式系统设计模式。