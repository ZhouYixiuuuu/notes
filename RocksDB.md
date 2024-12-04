# RocksDB

## 编译
`sudo make static_lib -j 8`

### 编译文件rocksdb_demo

```
g++ -std=c++17 rocksdb_demo.cpp -o rocksdb_demo librocksdb.a -I ./include -lpthread -ldl -lgflags -lz -lbz2 -lsnappy -llz4 -lzstd
```

### 使用db_bench
```
export PORTABLE=1
export CXXFLAGS=-fPIC
make clean
make db_bench
./db_bench --benchmarks=fillseq
```

使用load-gen

```
cd ../../K-V-Workload-Generator/ && ./load_gen -I1000000 -U1000000 -Q50000
cp workload.txt ../examples/__working_branch/ 

cd ../examples/__working_branch/ && make && ./working_version
```



## 事务

复习一下，四种常见的隔离级别：

* 读未提交：该隔离级别的事务可以看到其他事务中未提交的数据。该隔离级别因为可以读取到其他事务中未提交的数据，而未提交的数据可能会发生回滚，因此我们把该级别读取到的数据称之为脏数据，把这个问题称之为脏读。
* 读已提交：该隔离级别的事务能读取到已经提交事务的数据，因此它不会有脏读问题。但由于在事务的执行中可以读取到其他事务提交的结果，所以在不同时间的相同 SQL 查询中，可能会得到不同的结果，这种现象叫做不可重复读。
* 可重复读：所谓的幻读指的是，在同一事务的不同时间使用相同 SQL 查询时，会产生不同的结果。在可重复读隔离级别下，普通的查询操作（快照读）是不会看到其他事务插入的数据的。但是，对于会对数据进行修改的操作（如UPDATE、INSERT、DELETE），这些操作采用的是当前读模式。例如，在一个事务中，第一次更新可以成功更新，但此时另一个事务把这个数据删掉了，第二次更新就发现无法更新了。
* 可串行化：强制事务之间的执行有序。

RocksDB默认实现的是读已提交（使用write batch）和可重复读（快照隔离）。使用PessimisticTransaction。

![image-20241107162848945](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202411071630810.png)

![image-20241107163025864](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202411071630947.png)

* Put接口被调用后，会组织成一个write batch数据结构，在commit的时候原子提交。
* 每个KV对除了插入writeBatch，还插入（offset，不是完整的数据，而是在writeBatch中的偏移量）一个叫做write batch with index的数据结构中。这是一个跳表，用来提供事务操作过程中的read-your-write以及savepoint/rollback这样的功能。
* 事务如果没有提交，所有的数据都还没有写入到memtable中，都被缓存在WBWI中。每个事务都有一个独立的WBWI。

对于pessimisticTransaction实现：

* 在事务有更新操作的时候，对key进行独占。（TryLock）
* 从version系统中找到local_version，直接暴力遍历这个version中的mem/imm/sst，拿到当前key的最新的一个seq

参考这一篇：https://developer.aliyun.com/article/609664

## 写流程

![image-20241107165829489](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202411071658322.png)

## 读流程



几个问题：

### memtable怎么变成SSTable？

#### memtable的结构

![image-20241107175038971](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202411071750316.png)

参考一下这个

https://oi-wiki.org/ds/skiplist/

#### SSTable的结构

```
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1: filter block]                  (see section: "filter" Meta Block)
[meta block 2: index block]
[meta block 3: compression dictionary block]  (see section: "compression dictionary" Meta Block)
[meta block 4: range deletion block]          (see section: "range deletion" Meta Block)
[meta block 5: stats block]                   (see section: "properties" Meta Block)
...
[meta block K: future extended block]  (we may add more meta blocks in the future)
[metaindex block]
[Footer]                               (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```

使用sst_dump把sstable输出看一下，使用方法如下

https://github.com/facebook/rocksdb/wiki/Administration-and-Data-Access-Tool

```
sudo make sst_dump
sudo ./sst_dump --file=/tmp/rocksdb_demo/000028.sst --command=raw
这个命令是根据你数据库的位置
```

```
Footer Details:
--------------------------------------
  metaindex handle: EC0736 offset: 1004 size: 54
  index handle: 0000 offset: 0 size: 0
  table_magic_number: 9863518390377041911
  format version: 6

Metaindex Details:
--------------------------------------
  Properties block handle: 3FA807

Table Properties:
--------------------------------------
  # data blocks: 1
  # entries: 2
  # deletions: 0
  # merge operands: 0
  # range deletions: 0
  raw key size: 24
  raw average key size: 12.000000
  raw value size: 12
  raw average value size: 6.000000
  data block size: 42
  index block size (user-key? 1, delta-value? 1): 21
  filter block size: 0
  # entries for filter: 0
  (estimated) table size: 63
  filter policy name: N/A
  prefix extractor name: nullptr
  column family ID: 0
  column family name: default
  comparator name: leveldb.BytewiseComparator
  user defined timestamps persisted: true
  largest sequence number in file: 0
  merge operator name: nullptr
  property collectors names: []
  SST file compression algo: Snappy
  SST file compression options: window_bits=-14; level=32767; strategy=0; max_dict_bytes=0; zstd_max_train_bytes=0; enabled=0; max_dict_buffer_bytes=0; use_zstd_dict_trainer=1; 
  creation time: 1730960656
  time stamp of earliest key: 0
  time stamp of newest key: 1730980690
  file creation time: 1730980690
  slow compression estimated data size: 0
  fast compression estimated data size: 0
  DB identity: d9818400-d237-49ff-b609-ce3aece7f979
  DB session identity: 1FB8L6H8R07J06AIS2LC
  DB host id: iZf8ze1qcyph0c7dialkczZ
  original file number: 28
  unique ID: 2A8641B7A57CA20B-A177CB32D2FFA98B
  Sequence number to time mapping: 
  
Index Details:
--------------------------------------
  Block key hex dump: Data block handle
  Block key ascii

  HEX    6B657932: 0025 offset 0 size 37
  ASCII  k e y 2 
  ------

Data Block # 1 @ 0025
--------------------------------------
  HEX    6B657931: 76616C756531
  ASCII  k e y 1 : v a l u e 1 
  ------
  HEX    6B657932: 76616C756532
  ASCII  k e y 2 : v a l u e 2 
  ------

Data Block Summary:
--------------------------------------
  # data blocks: 1
  min data block size: 37
  max data block size: 37
  avg data block size: 37.000000
```

## compaction

compaction的线程是 BGWorkCompaction

这个线程在两种情况下会被调用

* 手动compact(RunManualCompaction)
* 自动(MaybeScheduleFlushOrCompaction)

![image-20241117130646509](C:/Users/z1382/AppData/Roaming/Typora/typora-user-images/image-20241117130646509.png)

MaybeScheduleFlushOrCompaction(db_impl_compaction_flush.cc)

类似flush的逻辑，compact的时候RocksDB也有一个队列叫做DBImpl::compaction_queue_.

```
  std::deque<ColumnFamilyData*> compaction_queue_;
```

然后我们来看这个队列何时被更新,其中unscheduled_compactions_和队列的更新是同步的，因此只有compaction_queue_更新之后，调用compact后台线程才会进入compact处理.

```c++
void DBImpl::EnqueuePendingCompaction(ColumnFamilyData* cfd) {
  mutex_.AssertHeld();
  if (reject_new_background_jobs_) {
    return;
  }
  if (!cfd->queued_for_compaction() && cfd->NeedsCompaction()) {
    TEST_SYNC_POINT_CALLBACK("EnqueuePendingCompaction::cfd",
                             static_cast<void*>(cfd));
    AddToCompactionQueue(cfd);
  }
}
```

分析一下`NeedsCompaction`函数是怎么样的？（空着先）

```

```

在MaybeScheduleFlushOrCompaction函数中，会调用BGWorkCompaction这个函数，代码如下：

先取出prepicked_compaction，但是暂时是nullptr

调用BackgroundCallCompaction

```c++
void DBImpl::BGWorkCompaction(void* arg) {
  CompactionArg ca = *(static_cast<CompactionArg*>(arg));
  delete static_cast<CompactionArg*>(arg);
  IOSTATS_SET_THREAD_POOL_ID(Env::Priority::LOW);
  TEST_SYNC_POINT("DBImpl::BGWorkCompaction");
  auto prepicked_compaction =
      static_cast<PrepickedCompaction*>(ca.prepicked_compaction);
  static_cast_with_check<DBImpl>(ca.db)->BackgroundCallCompaction(
      prepicked_compaction, Env::Priority::LOW);
  delete prepicked_compaction;
}
```

下面看看BackgroundCallCompaction，这个函数主要用于调度compaction任务，在compaction过程中遇到错误，能够正确地处理这些错误。记录compaction执行情况和错误信息，在compaction任务完成之后，识别并删除不再需要的临时文件，以释放存储空间。

下一个函数，调用BackgroundCompaction，这是最重要的函数了！

先看一下里面的PickCompaction函数：

```
c.reset(cfd->PickCompaction(*mutable_cf_options, log_buffer));
```

从上面可以看到PickCompaction这个函数，而这个函数会根据设置的不同的Compact策略调用不同的方法，这里我们只看默认的LevelCompact的对应函数.

```c++
Compaction* LevelCompactionBuilder::PickCompaction() {
  // Pick up the first file to start compaction. It may have been extended
  // to a clean cut.
  SetupInitialFiles();
  if (start_level_inputs_.empty()) {
    return nullptr;
  }
  assert(start_level_ >= 0 && output_level_ >= 0);

  // If it is a L0 -> base level compaction, we need to set up other L0
  // files if needed.
  if (!SetupOtherL0FilesIfNeeded()) {
    return nullptr;
  }

  // Pick files in the output level and expand more files in the start level
  // if needed.
  if (!SetupOtherInputsIfNeeded()) {
    return nullptr;
  }

  // Form a compaction object containing the files we picked.
  Compaction* c = GetCompaction();

  TEST_SYNC_POINT_CALLBACK("LevelCompactionPicker::PickCompaction:Return", c);

  return c;
}
```

先看第一个SetupInitialFiles函数：void LevelCompactionBuilder::SetupInitialFiles() 

BackgroundCompaction：

里面有edit()->DeleteFile将文件从旧一层中删除。





![image-20241117143243226](C:/Users/z1382/AppData/Roaming/Typora/typora-user-images/image-20241117143243226.png)

在RocksDB中，压缩任务可以根据其紧急性和对写入性能的影响被分配到**不同的后台线程池**中。

如果存在底部池（bottom pool），那么那些涉及最后一个级别的压缩任务（**通常是最有可能影响写入性能的任务**）将被调度到**低优先级**的底部池中。

这是一种优化策略，通过将某些压缩任务推迟或降低优先级，来保证数据库在高负载时的写入性能。

这一部分将compaction任务丢到低优先级后台线程池中。

```c++
env_->Schedule(&DBImpl::BGWorkBottomCompaction, ca, Env::Priority::BOTTOM,
                   this, &DBImpl::UnscheduleCompactionCallback);
```

下一步：进入最后一个else分支看看

首先创建一个CompactionJob

调用`compaction_job.Prepare()`：为压缩任务做好准备，如果需要划分为子任务，就计算子任务的边界，记录统计信息，处理时间信息等等。

调用`compaction_job.Run()`：多线程执行子压缩任务，状态更新以及验证压缩结果。

最终compact工作是在CompactionJob::ProcessKeyValueCompaction是实现的

![image-20241117183501725](C:/Users/z1382/AppData/Roaming/Typora/typora-user-images/image-20241117183501725.png)

在Compaction_job的run函数里面，在sstable验证完之后



关注几个函数

ECNetutils文件中的，

在compactionTask里面有个函数叫做RunMayThrow，里面有关于纠删sstable的Compaction

在columnFamilyStore中SendSSTRunnable

**Compaction的延迟在哪里啊？**

对于primary node来说：



一分钟进行一次getSendSSTRunnable（在ColumnFamilyStore中，在primary LSM-tree中），在最底下一层，先按过去两小时的访问频率进行排序，遍历所有sstable，如果还没有变成纠删编码，并且存在了足够长的时间(duration >= delayMilli)，下面就是调用syncSSTableWithSecondaryNodes通知次级节点，并发送ecMessage。

```
ScheduledExecutors.optionalTasks.scheduleWithFixedDelay(
ColumnFamilyStore.getSendSSTRunnable("ycsb", "usertable0", LeveledGenerations.getMaxLevelCount() - 1, DatabaseDescriptor.getTaskDelay()),
DatabaseDescriptor.getInitialDelay(),
1,
TimeUnit.MINUTES);
```



第二个，在Compaction的时候，在compactionTask里面有个函数叫做RunMayThrow，这个文件里面有超多RunMayThrow函数，找到一个（这个是针对secondary LSM-tree的），其中有openECSSTable，这一步也会有时间消耗。

加入针对（primary LSM-tree）的话，还会有上面这个时间消耗吗？不会有啊，这里的时间消耗就是一次Compaction完，就要新旧节点对比，然后通知次级节点和校验节点



在L0层到L1层，变成纠删码，L1-Ln是既有ECsstable，也有普通sstable

在Compaction的过程中，要算openECSStable的时间，在Compaction之后，生成新的sstable，加入等待变成EC的队列



更新：

在Compaction之后，生成新的sstable，加入等待变成EC的队列（但是这个过程是异步的。。）会对Compaction有消耗嘛。。。

实验应该做的是，在secondary LSM-tree里面的，在Compaction的时候从其他节点获取数据的开销。但是，在secondary LSM-tree中，Compaction之后会有发送数据的操作嘛？？



`openECSSTable` 函数本身并没有直接涉及从其他节点获取数据。这个函数主要执行以下操作：

1. **生成SSTableID**：为新的SSTable生成一个唯一的ID。
2. **目录操作**：获取数据目录，并根据提供的前缀找到对应的存储路径。
3. **文件移动**：将SSTable的组件文件（如 `Filter.db`, `Index.db`, `Statistics.db`, `Summary.db`）从重写目录移动到数据目录。
4. **TOC文件写入**：创建并写入TOC（Table of Contents）文件，该文件记录了SSTable的所有组件。
5. **加载ECMetadata**：加载与纠删编码相关的元数据。
6. **打开SSTable**：使用SSTable的描述符打开SSTable，并返回一个`SSTableReader`对象。



在ECRequestDataVerbHandler文件中的doVerb：以下是代码的详细解释和函数的作用：

1. **提取消息内容**：
   - 从传入的消息 `message` 中提取SSTable的哈希ID（`sstHash`）、索引（`index`）和请求的SSTable哈希ID（`requestSSTHash`）。
2. **获取列族存储和SSTable集合**：
   - 打开键空间 `"ycsb"` 和列族 `"usertable0"`，然后获取最高级别的SSTable集合。
3. **查找请求的SSTable**：
   - 遍历SSTable集合，查找与请求的哈希ID匹配的SSTable。
4. **处理数据迁移**：
   - 如果找到的SSTable已经迁移到云端（`isDataMigrateToCloud`），并且本地尚未下载（`!ECNetutils.getIsDownloaded(sstable.getSSTableHashID())`），则从云端重新加载原始数据。
   - 使用 `SSTableReader.loadRawDataFromCloud` 方法从云端加载数据，并使用 `CountDownLatch` 等待加载完成。
5. **处理下载冲突**：
   - 如果SSTable已经在下载中，使用重试逻辑等待下载完成。
6. **构建响应数据**：
   - 如果SSTable已下载并且可用，获取其内容，并构建 `ECResponseData` 响应消息。
7. **错误处理**：
   - 如果在遍历过程中没有找到请求的SSTable，抛出 `IllegalStateException` 异常。

在ECResponseDataVerbHandler中的doVerb函数，接受来自其他节点的纠删编码数据响应，以下是代码的详细解释和函数的作用：

1. **提取消息内容**：
   - 从传入的消息 `message` 中提取SSTable的哈希ID（`sstHash`）、原始数据（`rawData`）和索引（`index`）。
2. **检查全局映射**：
   - 检查 `StorageService.instance.globalSSTHashToErasureCodesMap` 全局映射中是否存在与请求的SSTable哈希ID（`sstHash`）对应的条目。
3. **验证索引范围**：
   - 验证提供的索引（`index`）是否在有效范围内，即是否在数据节点的数量范围内（`0` 到 `DatabaseDescriptor.getEcDataNodes() - 1`）。
4. **存储数据片段**：
   - 如果索引有效，并且全局映射中对应的位置已经有预分配的缓冲区，且缓冲区大小足以容纳接收到的原始数据，则将数据存储到该位置。
5. **记录日志**：
   - 记录调试信息，包括数据来源节点、SSTable哈希ID和索引。
6. **异常处理**：
   - 如果索引超出范围、全局映射中没有对应的条目，或者缓冲区大小不足以容纳数据，则抛出异常。
   - 抛出的异常包括 `IllegalStateException`（非法状态异常）和 `NullPointerException`（空指针异常）。



再重新捋一遍Compaction的流程：

1. 在columnFamilyStore中，有个**getBackgroundCompactionTaskSubmitter函数**，一分钟被调用一次，检查所有的columnFamily，对每个cf调用submitbackground

2. 在**submitbackground函数**中，检查当前cf是否有正在进行的Compaction任务，如果有正在进行的Compaction任务，并且执行器（`executor`）的活跃任务数已经达到了最大池大小，则记录一条trace级别的日志，并返回一个空的列表，表示当前不需要提交新的Compaction任务。

   如果没有，则创建一个 `BackgroundCompactionCandidate` 对象，它封装了Compaction任务的候选信息。使用 `executor.submitIfRunning` 方法提交后台Compaction任务。

   当你提交一个任务给`ExecutorService`执行时，它会返回一个`Future`对象，这个对象可以用来检查任务是否完成、等待任务完成或者取消任务。

   **在Java中，`Future`接口代表了一个可能还没有完成的异步计算的结果。**

3. 在名为 `BackgroundCompactionCandidate` 的内部类中，用于执行后台Compaction任务。`task = strategy.getNextBackgroundTask(getDefaultGcBefore(cfs, FBUtilities.nowInSeconds()));`

   * 首先，检查列族存储是否有效，如果已经被丢弃，则终止Compaction。
   * 获取Compaction策略管理器 `CompactionStrategyManager`，并尝试获取下一个后台Compaction任务。
   * 如果没有任务，并且开启了自动SSTable升级，则尝试执行升级任务。（升级任务是什么？？？）
   * 如果任务包含复制到纠删编码的SSTable，则记录调试信息，并执行任务。`task.execute(active, TransferredSSTableKeyRanges);`
   * 如果任务不包含复制到纠删编码的SSTable，则直接执行任务。
   * 无论Compaction是否成功，都从 `compactingCF` 集合（记录正在执行Compaction的cf）中移除当前的列族存储对象。

4. 进getNextBackgroundTask函数里面看看（在LeveledCompactionStrategy文件里面），它用于获取下一个要执行的后台Compaction任务。（循环地获取任务，直到成功为止）这个方法考虑了多种情况，包括标准的Compaction、基于可丢弃的墓碑（tombstone）比例的Compaction，以及涉及复制到纠删编码的SSTable的特殊处理。获取任务的函数是getCompactionCandidates：

   在这个函数中，在secondary LSM-tree中，对于已经变成纠删编码的sstable

   ```
   TransferredSSTableKeyRange range = new TransferredSSTableKeyRange(sstable.first, sstable.last);
   transferredSSTableKeyRanges.add(range);
   ```

   将转变的范围加进transferredSSTableKeyRanges。

   

5. 进入getCompactionCandidates中看看（终于找到了，这里，选择哪些sstable进行Compaction）：

   这里一开始先讲到了一个问题，

   * LevelDB为每个级别（level）计算一个得分，这个得分基于该级别中的数据量与其理想数据量的比例。得分最高的级别会被优先Compaction。
   * 但是会产生一个问题：例如，如果L0级别的理想数据量是4个SSTable，但实际上有988个，那么L0的得分会非常高（几乎250），而L1级别的得分可能只有11。为了处理这个问题，系统会批量处理L0级别的SSTable，与L1级别的所有SSTable一起Compaction，然后将结果（例如120个SSTable）放入L1级别。这个过程会重复进行，直到L0级别的SSTable被全部处理。
   * 由于内存限制，不能一次性将所有L0级别的SSTable与L1级别的SSTable一起Compaction。
   * LevelDB在L0级别的Compaction落后时会阻塞写入操作。但这段注释说明，当前系统没有这样的奢侈（即不能简单地阻塞写入）。

   针对这个问题，提出了两个解决方案：

   * **优先Compaction高级别**：首先强制对高级别的SSTable进行Compaction。这样做可以最小化Compaction所需的I/O操作，因为高级别通常包含更旧的数据，这些数据可能已经被覆盖或删除
   * **L0级别的Size Tiered Compaction（STCS）**：如果L0级别落后，系统会执行STCS以减少读取开销，直到赶上更高级别的Compaction。STCS是一种基于SSTable大小的Compaction策略，旨在减少读取时需要检查的SSTable数量，从而降低读取操作的I/O开销。

   整个函数流程就是：

   * 先获取L0级别的STCS Compaction candidates
   * 如果L0级别落后太多，现处理更高级别的Compaction
   * 从最高层开始遍历，计算分数，分数>1.001，返回当前层的sstable。对应的代码是`Collection<SSTableReader> candidates = getCandidatesForELECT(i);`,`int nextLevel = getNextLevel(candidates);`当前层和下一层一块返回。
   * 如果说高层没有需要Compaction的，那就返回`getCandidatesFor(0)`，non-STCS L0 compaction

   

6. 看一下getCandidatesForELECT这个函数

   先看里面有这样子两行代码，怎么理解？

   ```
   // 在primary LSM-tree中，变成纠删码了，没有被选为Compaction候选，就跳过
   if(cfs.getColumnFamilyName().equals("usertable0") && sstable.isReplicationTransferredToErasureCoding()) {
     if(!isSelectIssuedSSTableAsCompactionCandidates(sstable))
       continue;}
   // 正在被转变成纠删码/正在被Compaction，也跳过
   if(!sstable.isReplicationTransferredToErasureCoding() && ECNetutils.isSSTableCompactingOrErasureCoding(sstable.getSSTableHashID())) {
        continue;
   }
   ```

   后面再把下一层与当前层有重叠的sstable选出来，返回给上一个函数

   



几个runMayThrow函数，这几个都是CompactionTask extends AbstractCompactionTask中的

* `protected void runMayThrow(DecoratedKey first, DecoratedKey last, ECMetadata ecMetadata, String fileNamePrefix, Map<String, DecoratedKey> sourceKeys)`：这个是针对secondary LSM-tree的
* `protected void runMayThrow()`：这个是给primary LSM-tree使用的，但是这里说仅供internal use and testing only
* `protected void runMayThrow(List<TransferredSSTableKeyRange> TransferredSSTableKeyRanges)`：这个说在secondary node上使用，

再往外包一层是一个叫做WrappedRunnable的文件



在Compaction的时候跳过ECSSTable？？再定期去更新ECMetadata为新的ECSSTable？

这段代码定义了一个名为 `ConsumeBlockedECMetadataRunnable` 的内部类，它实现了 `Runnable` 接口，用于定期执行任务，消费阻塞的 ECMetadata（纠删编码元数据），将其转换为 ecSSTable（纠删编码固态表）。

这个函数又是一分钟被调用一次。。。。

transformECMetadataToECSSTable

transformECMetadataToECSSTableForErasureCode
这两啥关系啊？

* 第一个函数是通用接口：检查ECMetadata是否表示奇偶校验更新，如果是，则标记旧的SSTable为更新状态并调用`transformECMetadataToECSSTableForParityUpdate`。

  如果不是奇偶校验更新，则尝试从全局映射中获取重写数据，并调用`transformECMetadataToECSSTableForErasureCode`。

* 第二个函数就是小一点的接口



这个定期执行的流程捋一遍。。。

* ColumnFamilyStore.getSendSSTRunnable("ycsb", "usertable0", LeveledGenerations.getMaxLevelCount() - 1, DatabaseDescriptor.getTaskDelay())：只在primary LSM-tree中进行，定期将最底层sstable变成纠删编码

* ECMessageVerbHandler.getErasureCodingRunable()：这是校验节点计算纠删编码，它会设置一个全局接收队列，当队列中的数据块达到了一定的数量（DatabaseDescriptor.getEcDataNodes()），如果小于这个数，重试几次，如果重试次数用完了，并且等待队列不空，就用零填充；如果已经大于这个次数了，就进行纠删编码。

* ECMetadataVerbHandler.getConsumeBlockedECMetadataRunnable()：每个节点将ECMetadata存放在`globalReadyECMetadatas`的映射（Map）中，定期将ECMetadata转换为ecSSTable，即经过纠删编码处理的SSTable。ECMetadata可以在两种情况下生成：

  * 首次生成纠删码
  * 纠删码更新

  ```
  // 这是需要被更新的sstable
  List<SSTableReader> sstables = new ArrayList<>(
  cfs.getSSTableForLevel(LeveledGenerations.getMaxLevelCount() - 1));
  // 最后一层的sstable
  List<SSTableReader> rewriteSStables = new ArrayList<SSTableReader>(
  LeveledManifest.overlapping(firstKeyForRewrite.getToken(),
                                      lastKeyForRewrite.getToken(),
                                      sstables));
  ```

  在这个函数中调用了transformECMetadataToECSSTableForErasureCode函数，点进去看看：

  一开始，先封装了SSTable修改操作的事务对象。它管理着压缩过程中的SSTable状态，确保操作的原子性和一致性。

  ```
  final LifecycleTransaction updateTxn = cfs.getTracker().tryModify(rewriteSStables, OperationType.COMPACTION);
  ```

  

  在里面又调用了sstablesRewrite函数：调用了performSSTableRewrite

  直接执行一个Compactiontask

  ```
  AbstractCompactionTask task = cfs.getCompactionStrategyManager().getCompactionTask(txn, NO_GC,
               Long.MAX_VALUE);
  task.execute(active, first, last, ecMetadata, fileNamePrefix, sourceKeys);
  ```

  

  forceCompactionForTokenRange看看这个是啥？？？？

  

  

  

  现在的思路是，在Compaction的时候，

  存什么数据在队列里面，

## Interator Internal 

汇总 memtable + 多个 sst level 的数据





```
bool for_compaction = caller == TableReaderCaller::kCompaction;
  auto& fd = file_meta.fd;
  table_reader = fd.table_reader;
  if (table_reader == nullptr) {
    s = FindTable(options, file_options, icomparator, file_meta, &handle,
                  mutable_cf_options,
                  options.read_tier == kBlockCacheTier /* no_io */,
                  file_read_hist, skip_filters, level,
                  true /* prefetch_index_and_filter_in_cache */,
                  max_file_size_for_l0_meta_pin, file_meta.temperature);
    if (s.ok()) {
      table_reader = cache_.Value(handle);
    }
  }
```





## db_bench

```
./db_bench --benchmarks=fillseq,stats --num=2314099 --compression_type=none --compaction_style=0

./db_bench --benchmarks=fillrandom,stats --num=4628198 --compression_type=none --max_bytes_for_level_multiplier=2 --compaction_style=0

./db_bench --benchmarks="overwrite,stats" --use_existing_db=true
```

![image-20241126104252298](C:/Users/z1382/AppData/Roaming/Typora/typora-user-images/image-20241126104252298.png)

https://github.com/facebook/rocksdb/wiki/RocksDB-Tuning-Guide

level_compaction_dynamic_level_bytes：

* true：

一共有几种不同的Compaction策略：

* Full：?? 待实现？？？rocksdb没有实现~~

  ```
  // 每张sstable大小都一样
  ./db_bench --benchmarks=fillrandom,stats --num=4628198 --compression_type=none --max_bytes_for_level_multiplier=2 --compaction_style=0 
  ```

  

* LO+1：

  compaction_pri默认是3

  ```c++
  enum CompactionPri : char {
    // Slightly prioritize larger files by size compensated by #deletes
    kByCompensatedSize = 0x0,
    // First compact files whose data's latest update time is oldest.
    // Try this if you only update some hot keys in small ranges.
    kOldestLargestSeqFirst = 0x1,
    // First compact files whose range hasn't been compacted to the next level
    // for the longest. If your updates are random across the key space,
    // write amplification is slightly better with this option.
    kOldestSmallestSeqFirst = 0x2,
    // First compact files whose ratio between overlapping size in next level
    // and its size is the smallest. It in many cases can optimize write
    // amplification.
    // Files marked for compaction will be prioritized over files that are not
    // marked.
    kMinOverlappingRatio = 0x3,
    // Keeps a cursor(s) of the successor of the file (key range) was/were
    // compacted before, and always picks the next files (key range) in that
    // level. The file picking process will cycle through all the files in a
    // round-robin manner.
    kRoundRobin = 0x4,
  };
  ```

  

  ```
  ./db_bench --benchmarks=fillrandom,stats --num=4628198 --compaction_style=0 --compaction_pri=3 --max_bytes_for_level_multiplier=2
  ```

  

* Cold：

  ```
  ./db_bench --benchmarks=fillrandom,stats --num=4628198 --compaction_style=0 --compaction_pri=1 --max_bytes_for_level_multiplier=2
  ```

  

* Old：

  ```
  ./db_bench --benchmarks=fillrandom,stats --num=4628198 --compaction_style=0 --compaction_pri=2 --max_bytes_for_level_multiplier=2
  ```

  

* TSD：

  NewCompactOnDeletionCollectorFactory(kWindowSize, kNumDelsTrigger)

  * `sliding_window_size "N"`：这是一个滑动窗口的大小，用于检查连续的条目。如果在这个窗口中发现足够的删除条目，就会触发压缩。注意，这个数值会被向上取整到不小于指定大小的最小的128的倍数。
  * `deletion_trigger "D"`：这是触发压缩的删除条目的最小数量。即使改变了N的值，D的指定数量也不会改变。
  * `deletion_ratio`：这是一个比率阈值，用于基于墓碑条目的比例触发压缩。如果这个值小于或等于0，或者大于1，则基于删除比率的压缩触发会被禁用。默认情况下是禁用的。

  ```
  ./db_bench --benchmarks=fillrandom,stats --num=4628198 --compaction_style=0 --compaction_pri=0 --max_bytes_for_level_multiplier=2 
  ```

  使用 kByCompensatedSize 策略时，RocksDB 会优先选择包含较多 tombstone(删除标记)的文件进行 compaction。具体逻辑是:

  1. 计算每个文件的 compensated file size:

  - compensated_file_size = 实际文件大小 + (删除数量 - 插入数量) * 某个因子

  1. 当一个文件中删除数量超过插入数量时:

  - compensated_file_size 会变大
  - 该文件被选中进行 compaction 的优先级就会提高

  1. 删除数量比插入数量超出越多:

  - compensated_file_size 越大
  - 被选中的优先级越高

* RR：

  ```
  ./db_bench --benchmarks=fillrandom,stats --num=4628198 --compaction_style=0 --compaction_pri=4 --max_bytes_for_level_multiplier=2
  ```

  

* LO+2

* TSA

* Tier：



## 复制sstable

```c++
Status TableCache::GetTableReader(
    const ReadOptions& ro, const FileOptions& file_options,
    const InternalKeyComparator& internal_comparator, const FileDescriptor& fd,
    bool sequential_mode, bool record_read_stats, HistogramImpl* file_read_hist,
    std::unique_ptr<TableReader>* table_reader,
    const SliceTransform* prefix_extractor, bool skip_filters, int level,
    bool prefetch_index_and_filter_in_cache,
    size_t max_file_size_for_l0_meta_pin) {
  std::string fname =
      TableFileName(ioptions_.cf_paths, fd.GetNumber(), fd.GetPathId());
  std::unique_ptr<FSRandomAccessFile> file;
  FileOptions fopts = file_options;
  Status s = PrepareIOFromReadOptions(ro, ioptions_.env, fopts.io_options);
  if (s.ok()) {
    s = ioptions_.fs->NewRandomAccessFile(fname, fopts, &file, nullptr);
  }
  RecordTick(ioptions_.statistics, NO_FILE_OPENS);
  if (s.IsPathNotFound()) {
    fname = Rocks2LevelTableFileName(fname);
    s = PrepareIOFromReadOptions(ro, ioptions_.env, fopts.io_options);
    if (s.ok()) {
      s = ioptions_.fs->NewRandomAccessFile(fname, file_options, &file,
                                            nullptr);
    }
    RecordTick(ioptions_.statistics, NO_FILE_OPENS);
  }

  if (s.ok()) {
    if (!sequential_mode && ioptions_.advise_random_on_open) {
      file->Hint(FSRandomAccessFile::kRandom);
    }
    StopWatch sw(ioptions_.env, ioptions_.statistics, TABLE_OPEN_IO_MICROS);
    std::unique_ptr<RandomAccessFileReader> file_reader(
        new RandomAccessFileReader(
            std::move(file), fname, ioptions_.env,
            record_read_stats ? ioptions_.statistics : nullptr, SST_READ_MICROS,
            file_read_hist, ioptions_.rate_limiter, ioptions_.listeners));
    s = ioptions_.table_factory->NewTableReader(
        ro,
        TableReaderOptions(ioptions_, prefix_extractor, file_options,
                           internal_comparator, skip_filters, immortal_tables_,
                           false /* force_direct_prefetch */, level,
                           fd.largest_seqno, block_cache_tracer_,
                           max_file_size_for_l0_meta_pin),
        std::move(file_reader), fd.GetFileSize(), table_reader,
        prefetch_index_and_filter_in_cache);
    TEST_SYNC_POINT("TableCache::GetTableReader:0");
  }
  return s;
}
```





```c++
Status CompactionJob::OpenCompactionOutputFile(
    SubcompactionState* sub_compact) {
  assert(sub_compact != nullptr);
  assert(sub_compact->builder == nullptr);
  // no need to lock because VersionSet::next_file_number_ is atomic
  uint64_t file_number = versions_->NewFileNumber();
  std::string fname =
      TableFileName(sub_compact->compaction->immutable_cf_options()->cf_paths,
                    file_number, sub_compact->compaction->output_path_id());
  // Fire events.
  ColumnFamilyData* cfd = sub_compact->compaction->column_family_data();
#ifndef ROCKSDB_LITE
  EventHelpers::NotifyTableFileCreationStarted(
      cfd->ioptions()->listeners, dbname_, cfd->GetName(), fname, job_id_,
      TableFileCreationReason::kCompaction);
#endif  // !ROCKSDB_LITE
  // Make the output file
  std::unique_ptr<FSWritableFile> writable_file;
#ifndef NDEBUG
  bool syncpoint_arg = file_options_.use_direct_writes;
  TEST_SYNC_POINT_CALLBACK("CompactionJob::OpenCompactionOutputFile",
                           &syncpoint_arg);
#endif
  Status s;
  IOStatus io_s =
      NewWritableFile(fs_.get(), fname, &writable_file, file_options_);
  s = io_s;
  if (sub_compact->io_status.ok()) {
    sub_compact->io_status = io_s;
  }
  if (!s.ok()) {
    ROCKS_LOG_ERROR(
        db_options_.info_log,
        "[%s] [JOB %d] OpenCompactionOutputFiles for table #%" PRIu64
        " fails at NewWritableFile with status %s",
        sub_compact->compaction->column_family_data()->GetName().c_str(),
        job_id_, file_number, s.ToString().c_str());
    LogFlush(db_options_.info_log);
    EventHelpers::LogAndNotifyTableFileCreationFinished(
        event_logger_, cfd->ioptions()->listeners, dbname_, cfd->GetName(),
        fname, job_id_, FileDescriptor(), kInvalidBlobFileNumber,
        TableProperties(), TableFileCreationReason::kCompaction, s);
    return s;
  }

  // Try to figure out the output file's oldest ancester time.
  int64_t temp_current_time = 0;
  auto get_time_status = env_->GetCurrentTime(&temp_current_time);
  // Safe to proceed even if GetCurrentTime fails. So, log and proceed.
  if (!get_time_status.ok()) {
    ROCKS_LOG_WARN(db_options_.info_log,
                   "Failed to get current time. Status: %s",
                   get_time_status.ToString().c_str());
  }
  uint64_t current_time = static_cast<uint64_t>(temp_current_time);
  uint64_t oldest_ancester_time =
      sub_compact->compaction->MinInputFileOldestAncesterTime();
  if (oldest_ancester_time == port::kMaxUint64) {
    oldest_ancester_time = current_time;
  }

  // Initialize a SubcompactionState::Output and add it to sub_compact->outputs
  {
    SubcompactionState::Output out;
    out.meta.fd = FileDescriptor(file_number,
                                 sub_compact->compaction->output_path_id(), 0);
    out.meta.oldest_ancester_time = oldest_ancester_time;
    out.meta.file_creation_time = current_time;
    out.finished = false;
    out.paranoid_hash = 0;
    sub_compact->outputs.push_back(out);
  }

  writable_file->SetIOPriority(Env::IOPriority::IO_LOW);
  writable_file->SetWriteLifeTimeHint(write_hint_);
  writable_file->SetPreallocationBlockSize(static_cast<size_t>(
      sub_compact->compaction->OutputFilePreallocationSize()));
  const auto& listeners =
      sub_compact->compaction->immutable_cf_options()->listeners;
  sub_compact->outfile.reset(
      new WritableFileWriter(std::move(writable_file), fname, file_options_,
                             env_, db_options_.statistics.get(), listeners,
                             db_options_.file_checksum_gen_factory.get()));

  // If the Column family flag is to only optimize filters for hits,
  // we can skip creating filters if this is the bottommost_level where
  // data is going to be found
  bool skip_filters =
      cfd->ioptions()->optimize_filters_for_hits && bottommost_level_;

  sub_compact->builder.reset(NewTableBuilder(
      *cfd->ioptions(), *(sub_compact->compaction->mutable_cf_options()),
      cfd->internal_comparator(), cfd->int_tbl_prop_collector_factories(),
      cfd->GetID(), cfd->GetName(), sub_compact->outfile.get(),
      sub_compact->compaction->output_compression(),
      0 /*sample_for_compression */,
      sub_compact->compaction->output_compression_opts(),
      sub_compact->compaction->output_level(), skip_filters,
      oldest_ancester_time, 0 /* oldest_key_time */,
      sub_compact->compaction->max_output_file_size(), current_time, db_id_,
      db_session_id_));
  LogFlush(db_options_.info_log);
  return s;
}
```



```
const std::string kGood = "this is a good string, synced to disk";
  const std::string kCorrupted = "this part may be corrupted";
  const std::string kFileName = "/dir/f";
  std::unique_ptr<WritableFile> writable_file;
  ASSERT_OK(env_->NewWritableFile(kFileName, &writable_file, soptions_));
  ASSERT_OK(writable_file->Append(kGood));
  ASSERT_TRUE(writable_file->GetFileSize() == kGood.size());

  std::string scratch;
  scratch.resize(kGood.size() + kCorrupted.size() + 16);
  Slice result;
  std::unique_ptr<RandomAccessFile> rand_file;
  ASSERT_OK(env_->NewRandomAccessFile(kFileName, &rand_file, soptions_));
  ASSERT_OK(rand_file->Read(0, kGood.size(), &result, &(scratch[0])));
  ASSERT_EQ(result.compare(kGood), 0);

  // Sync + corrupt => no change
  ASSERT_OK(writable_file->Fsync());
  ASSERT_OK(dynamic_cast<MockEnv*>(env_)->CorruptBuffer(kFileName));
  result.clear();
  ASSERT_OK(rand_file->Read(0, kGood.size(), &result, &(scratch[0])));
  ASSERT_EQ(result.compare(kGood), 0);
```



有没有什么办法，通过FileMetaData或者FileMetaData.fileDescription获取table_properties呢？？

下面这段参考一下？？GetTableProperties这个函数

```c++
void DBImpl::BuildCompactionJobInfo(
    const ColumnFamilyData* cfd, Compaction* c, const Status& st,
    const CompactionJobStats& compaction_job_stats, const int job_id,
    const Version* current, CompactionJobInfo* compaction_job_info) const {
  assert(compaction_job_info != nullptr);
  compaction_job_info->cf_id = cfd->GetID();
  compaction_job_info->cf_name = cfd->GetName();
  compaction_job_info->status = st;
  compaction_job_info->thread_id = env_->GetThreadID();
  compaction_job_info->job_id = job_id;
  compaction_job_info->base_input_level = c->start_level();
  compaction_job_info->output_level = c->output_level();
  compaction_job_info->stats = compaction_job_stats;
  compaction_job_info->table_properties = c->GetOutputTableProperties();
  compaction_job_info->compaction_reason = c->compaction_reason();
  compaction_job_info->compression = c->output_compression();
  for (size_t i = 0; i < c->num_input_levels(); ++i) {
    for (const auto fmd : *c->inputs(i)) {
      const FileDescriptor& desc = fmd->fd;
      const uint64_t file_number = desc.GetNumber();
      auto fn = TableFileName(c->immutable_cf_options()->cf_paths, file_number,
                              desc.GetPathId());
      compaction_job_info->input_files.push_back(fn);
      compaction_job_info->input_file_infos.push_back(CompactionFileInfo{
          static_cast<int>(i), file_number, fmd->oldest_blob_file_number});
      if (compaction_job_info->table_properties.count(fn) == 0) {
        std::shared_ptr<const TableProperties> tp;
        auto s = current->GetTableProperties(&tp, fmd, &fn);
        if (s.ok()) {
          compaction_job_info->table_properties[fn] = tp;
        }
      }
    }
  }
  for (const auto& newf : c->edit()->GetNewFiles()) {
    const FileMetaData& meta = newf.second;
    const FileDescriptor& desc = meta.fd;
    const uint64_t file_number = desc.GetNumber();
    compaction_job_info->output_files.push_back(TableFileName(
        c->immutable_cf_options()->cf_paths, file_number, desc.GetPathId()));
    compaction_job_info->output_file_infos.push_back(CompactionFileInfo{
        newf.first, file_number, meta.oldest_blob_file_number});
  }
}
```





参加一下，不知道能不能用

```
void DBTestBase::CopyFile(const std::string& source,
                          const std::string& destination, uint64_t size) {
  const EnvOptions soptions;
  std::unique_ptr<SequentialFile> srcfile;
  ASSERT_OK(env_->NewSequentialFile(source, &srcfile, soptions));
  std::unique_ptr<WritableFile> destfile;
  ASSERT_OK(env_->NewWritableFile(destination, &destfile, soptions));

  if (size == 0) {
    // default argument means copy everything
    ASSERT_OK(env_->GetFileSize(source, &size));
  }

  char buffer[4096];
  Slice slice;
  while (size > 0) {
    uint64_t one = std::min(uint64_t(sizeof(buffer)), size);
    ASSERT_OK(srcfile->Read(one, &slice, buffer));
    ASSERT_OK(destfile->Append(slice));
    size -= slice.size();
  }
  ASSERT_OK(destfile->Close());
}
```



还有一个，参考一下：

```c++
IOStatus CopyFile(FileSystem* fs, const std::string& source,
                  const std::string& destination, uint64_t size,
                  bool use_fsync) {
  const FileOptions soptions;
  IOStatus io_s;
  std::unique_ptr<SequentialFileReader> src_reader;
  std::unique_ptr<WritableFileWriter> dest_writer;

  {
    std::unique_ptr<FSSequentialFile> srcfile;
    io_s = fs->NewSequentialFile(source, soptions, &srcfile, nullptr);
    if (!io_s.ok()) {
      return io_s;
    }
    std::unique_ptr<FSWritableFile> destfile;
    io_s = fs->NewWritableFile(destination, soptions, &destfile, nullptr);
    if (!io_s.ok()) {
      return io_s;
    }

    if (size == 0) {
      // default argument means copy everything
      io_s = fs->GetFileSize(source, IOOptions(), &size, nullptr);
      if (!io_s.ok()) {
        return io_s;
      }
    }
    src_reader.reset(new SequentialFileReader(std::move(srcfile), source));
    dest_writer.reset(
        new WritableFileWriter(std::move(destfile), destination, soptions));
  }

  char buffer[4096];
  Slice slice;
  while (size > 0) {
    size_t bytes_to_read = std::min(sizeof(buffer), static_cast<size_t>(size));
    io_s = status_to_io_status(src_reader->Read(bytes_to_read, &slice, buffer));
    if (!io_s.ok()) {
      return io_s;
    }
    if (slice.size() == 0) {
      return IOStatus::Corruption("file too small");
    }
    io_s = dest_writer->Append(slice);
    if (!io_s.ok()) {
      return io_s;
    }
    size -= slice.size();
  }
  return dest_writer->Sync(use_fsync);
}

extern IOStatus CopyFile(FileSystem* fs, const std::string& source,
                         const std::string& destination, uint64_t size,
                         bool use_fsync);  // 感觉可以直接引用？？？
// 用法示例
return CopyFile(db_->GetFileSystem(), src_dirname + fname,
                          full_private_path + fname, size_limit_bytes,
                          db_options.use_fsync);
```



删文件的函数。。

```
ASSERT_OK(env_->DeleteFile(fpath));
```



输入一个FileMetaData，返回一个FileMetaData

如何根据FileMetaData获知sstable的数据，比如data size？index size？或者将整个sstable变成纠删码？？

抄一下！！

```c++
Status SstFileDumper::GetTableReader(const std::string& file_path) {
  // Warning about 'magic_number' being uninitialized shows up only in UBsan
  // builds. Though access is guarded by 's.ok()' checks, fix the issue to
  // avoid any warnings.
  uint64_t magic_number = Footer::kInvalidTableMagicNumber;

  // read table magic number
  Footer footer;

  std::unique_ptr<RandomAccessFile> file;
  uint64_t file_size = 0;
  Status s = options_.env->NewRandomAccessFile(file_path, &file, soptions_);
  if (s.ok()) {
    s = options_.env->GetFileSize(file_path, &file_size);
  }

  // check empty file
  // if true, skip further processing of this file
  if (file_size == 0) {
    return Status::Aborted(file_path, "Empty file");
  }

  file_.reset(new RandomAccessFileReader(NewLegacyRandomAccessFileWrapper(file),
                                         file_path));

  FilePrefetchBuffer prefetch_buffer(nullptr, 0, 0, true /* enable */,
                                     false /* track_min_offset */);
  if (s.ok()) {
    const uint64_t kSstDumpTailPrefetchSize = 512 * 1024;
    uint64_t prefetch_size = (file_size > kSstDumpTailPrefetchSize)
                                 ? kSstDumpTailPrefetchSize
                                 : file_size;
    uint64_t prefetch_off = file_size - prefetch_size;
    IOOptions opts;
    prefetch_buffer.Prefetch(opts, file_.get(), prefetch_off,
                             static_cast<size_t>(prefetch_size));

    s = ReadFooterFromFile(opts, file_.get(), &prefetch_buffer, file_size,
                           &footer);
  }
  if (s.ok()) {
    magic_number = footer.table_magic_number();
  }

  if (s.ok()) {
    if (magic_number == kPlainTableMagicNumber ||
        magic_number == kLegacyPlainTableMagicNumber) {
      soptions_.use_mmap_reads = true;
      options_.env->NewRandomAccessFile(file_path, &file, soptions_);
      file_.reset(new RandomAccessFileReader(
          NewLegacyRandomAccessFileWrapper(file), file_path));
    }
    options_.comparator = &internal_comparator_;
    // For old sst format, ReadTableProperties might fail but file can be read
    if (ReadTableProperties(magic_number, file_.get(), file_size,
                            (magic_number == kBlockBasedTableMagicNumber)
                                ? &prefetch_buffer
                                : nullptr)
            .ok()) {
      SetTableOptionsByMagicNumber(magic_number);
    } else {
      SetOldTableOptions();
    }
  }

  if (s.ok()) {
    s = NewTableReader(ioptions_, soptions_, internal_comparator_, file_size,
                       &table_reader_);
  }
  return s;
}
```



在copyfile的时候，会将file放进versionstorageInfo里面吗？？？



## Direte IO

直接I/O（Direct IO）概念：

- **传统I/O**：当应用程序请求读或写磁盘时，操作系统会将数据缓存到内存中。这意味着后续相同数据的访问会更快，因为它们已经存在于内存中。然而，内存管理和缓存机制会消耗一定的系统资源和时间。
- **Direct IO**：在使用直接I/O时，数据直接从磁盘读取到应用程序提供的内存缓冲区（或者反过来写入磁盘）。操作系统的文件系统缓存被绕过。这意味着数据不会被存储在操作系统的页缓存中，也就减少了内存使用和操作系统缓存管理的开销。

## 实验命令

```
--memory_size=16777216  一张SSTable64MB

```

