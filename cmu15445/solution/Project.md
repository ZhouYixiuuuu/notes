# Project_solution

## P0 C++ Primer

### c++ 相关知识补充

1. `dynamic_cast`：父类转换成子类时使用，有安全机制保护

2. `std::dynamic_pointer_cast`：智能指针类型转换（不适用于`unique_ptr`

3. `const`的`map`访问元素只能用`at`，`[]`不适用！

4. 普通指针和智能指针的转换：

   ```c++
   int x = 10;
   int *ptr = &x;
   std::shared_ptr<int> sh_ptr = std::make_shared<int>(ptr);
   auto t = sh_ptr.get();
   ```

5. `auto ptr = std::shared_ptr<TrieNode>()`：调用`TrieNode`默认构造函数，创造出来的是`nullptr`

### Put()

1. `Clone()`可以解掉`const`
2. 但是要注意是否存在根节点！
3. 最后一个结点，直接新建一个`TrieNodeWithValue`，如果使用`Clone`创建出来的还是`TrieNode`

### Remove()

1. 在回溯时，遇到类型为`TrieNode`并且没有子节点的节点，都要删掉

![image-20240324164657862](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403241647336.png)

## P1 Buffer Pool

### Task #1 - LRU-K Replacement Policy

建两条队列，历史队列和缓存队列

#### Evict

* 判断历史队列是否为空
  * 不为空，就在历史队列中找到一个evictable frame
  * 找不到，就去缓存队列中找
  * 找到了，将它从历史队列中驱逐
* 在缓存队列中找
  * 找到evictable frame，将其驱逐
  * 找不到，return false

#### RecordAccess

* 在历史队列中，因为是FIFO策略，直接count ++
  * 如果count = k，就挪到缓存队列的头部
* 在缓存队列中，因为是LRU策略，直接挪到队列头部
* 如果都不在，新建一个节点，加入历史队列中

### Task #2 - Buffer Pool Manager

这里难度在于理解page和frame，两者要区分开来，frame是缓存池中的概念，可以理解为一个容器，里面装了page。

#### FetchPage

函数 `FetchPage`，就是我要得到一个页面，缓冲池中有，就从缓冲池中拿。没有就要从disk里面调取。

拿这个页面，就说明我用到了这个页面，LRUK队列中的accessCount就要加一，同时，也要被pin住，等到它用完（unpin），才可以从缓存池中离开。

每次发生驱逐，就要判断被驱逐的页面是否dirty，dirty就要写回disk

* 缓存池中没有
  * 从free_list中读取（缓存池是否有空位置）
  * 如何缓存池满了，就调用Evict，驱逐一些页面
  * 这上面两个，得到的都是frame，就是得到一个缓存池的位置
  * 再把新的page从disk读出来，写入这个frame的位置

#### NewPage

这个函数一开始，真的完全不能理解，它其实就是创建一个新的page（空page，不用从disk读取），然后加入缓存池

其实跟FetchPage一模一样。只是一个要读disk，一个不用而已。

### Task #3 - Read/Write Page Guards

![image-20240330141906421](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403301419520.png)

需要注意的是重置=

```c++
auto read_guard1 = bpm->FetchPageRead(page_id_temp);
auto read_guard2 = bpm->FetchPageRead(page_id_temp_a);
EXPECT_EQ(2, page0->GetPinCount());
EXPECT_EQ(2, page1->GetPinCount());
read_guard2 = std::move(read_guard1);
EXPECT_EQ(2, page0->GetPinCount());
EXPECT_EQ(1, page1->GetPinCount());
```

两个guard指向不同的页面，画个等号会发生什么？

* 前一个guard会drop，因为已经用完了
* 然后等于第二个guard

### bugs

* 缓存池满，死锁？缓存队列初始化有bug

* `LeakSanitizer: detected memory leaks`：在析构函数中，将所有LRUNode释放，就解决了  

* `LRUKReplacerTest.ConcurrencyTest`

  ```sh
  terminate called after throwing an instance of 'std::runtime_error'
  21:   what():  Remove ERROR
  ```

  仔细读题，未找到的frame直接返回，不用throw exception

* `PageGuardTest.MoveTest`

  ```sh
  terminate called after throwing an instance of 'std::system_error'
  64:   what():  Resource deadlock avoided
  ```

  在重置运算符=时，也要有释放锁的操作

* `BufferPoolManagerTest.FetchPage (2/2)`：在`FetchPage`里面没有`access and pin_count_++`

## P2 B+Tree

### Task #1 - B+Tree Pages

```
 * Leaf page format (keys are stored in order):
 *  ----------------------------------------------------------------------
 * | HEADER | KEY(1) + RID(1) | KEY(2) + RID(2) | ... | KEY(n) + RID(n)
 *  ----------------------------------------------------------------------
 *
 *  Header format (size in byte, 16 bytes in total):
 *  ---------------------------------------------------------------------
 * | PageType (4) | CurrentSize (4) | MaxSize (4) |
 *  ---------------------------------------------------------------------
 *  -----------------------------------------------
 * |  NextPageId (4)
 *  -----------------------------------------------
```

* 这里展示了LeafPage的结构，不能往BPlusTreePage中加新的私有（或共有）变量，会影响`array_`的大小，会报`Segmentation fault`

* 对`max_size`和`min_size`的理解

  什么时候`split`？

  * `leaf_size >= leaf_max_size`
  * `internal_size > internal_max_size`，不能有等号，这是不一样的

  什么时候`merge`？

  * `leaf_size < ceil(leaf_max_size / 2)`
  * `internal_size < ceil(internal_max_size / 2)`

  可以将`min_size`设置成为上取整。

### Insertion

思路：

* 一开始没有思路，可以模仿下面的`PrintTree(page_id_t page_id, const BPlusTreePage *page)`函数

* 设置一些子函数

  * `SplitLeafPage(WritePageGuard *guard, Context &ctx)`：`WritePageGuard`是后面上锁的时候加的，在给分点1中，可以写成`BPlusTreePage`，函数的作用就是分隔叶子结点
  * 同理设置一个 `SplitInternalPage(WritePageGuard *guard, Context &ctx)`，分隔中间节点
  * 分隔后的结点，要加入父节点。设置`InsertIntoParent(WritePageGuard *old_guard, WritePageGuard *new_guard, Context &ctx)`，在这个函数中，有两个需要注意的点！
    * 如果`old_guard`存的`old_page`是`root`，就说明，我们需要创建一个新的叶子结点（因为`new_page`是从`old_page`中分隔出来的，此时没有根节点了）
    * **需要先将`old_page_id`从父节点中删除**！再重新往父节点写入`new_page`和`old_page`，这个bug在`BPlusTreeTestsCP2.SequentialMixTest`和`BPlusTreeConcurrentTest.MixTest1`有所测试。对于本地的`b_plus_tree_sequential_scale_test`无法测出来，可以试着将B+树的`leaf_max_size = 3; internal_max_size = 5`

* 在这里使用了`ctx.write_set_`的话，当时遇到了几个bug

  * 某个`page`已经是`writeguard`了，再去`fetch`就会死锁（对`page_guard`的理解不过深入）

  * `deque`的`pop_back`的destroy只有析构对象的作用，没有deallocate的回收空间的作用。所以，pop_back之后，**空间是没有释放的**

    ![image-20240403171128535](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202404031711182.png)

    ```c++
    std::deque<Student> d;
    d.push_back(Student(13, "Alice"));
    
    Student * ptr = &(d.back());
    d.pop_back();
    std::cout << ptr->age;  // 不会报错，内存没有释放
    std::cout << d[0].age;  // 不会报错
    ```

    就是说，调用了`pop_back`函数，仍然存在一个指针指向这个`writeguard`，所以pop之后还要drop

    ![image-20240415194512365](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202404151945455.png)

    看一下文档怎么写的，当时没注意（主要也是不理解），所以报错了。或者可以这样写：

    ```c++
    auto parent_guard = std::move(ctx.write_set_.back());
    ctx.write_set_.pop_back();
    ```

    `std::move()`会解掉所有引用！！		

### An Iterator for Leaf Scans

要完成给分点1，还需要完成这个Task

* **注意理解**：`End() `返回的是一个空的对象，而不是最后一个KV

### Remove

不理解如何`Remove`可以先看看这篇文章

[CMU 15445-2022 P2 B+Tree Insert/Delete]: https://zhuanlan.zhihu.com/p/592964493

**一定要理解的点**：`merge`和`Redistribute`的结点，一定是自己的兄弟节点！（**有共同的父节点**），至于选择左节点还是右节点，都可以！

写几个子函数

* `RedistributeLeft(WritePageGuard *left_guard, WritePageGuard *guard, Context &ctx)`，和左边结点重构
* `RedistributeRight(WritePageGuard *right_guard, WritePageGuard *guard, Context &ctx)`，和右边结点重构
* `Merge(WritePageGuard *new_guard, WritePageGuard *old_guard, Context &ctx)`，将`old_page`与`new_page`合并，合并之后要调用`RemoveInternal`，将`old_page`从父节点中删除
* `RemoveInternal(page_id_t page_id, Context &ctx)`，从`ctx.write_set_.back()`取出父节点，将`page_id`删掉

**bugs**：

* 如果根节点只有一个子节点，要重新设置根节点
* 如果删成空树，记得重新设置`header page`

### Concurrent Index

**Crabbing Protocol（蟹行协议）**

1. 获取父节点的锁
2. 获取子节点的锁
3. 判断当前节点是否安全，安全则释放父节点的锁
   * insert操作：当前节点还有空位就还是安全的（不会split）
   * remove操作：当前节点半满则是安全的（不会merge）
   * getValue操作：一直安全

根据蟹行协议进行上锁，这里记录几个遇到的bug

* 在`Insert/Remove`中，**第一步，一定一定要先锁住`HeaderPage`**，除非判断`root page`安全，不要给`HeaderPage`解锁！

* 在`split/merge/redistribute`之后，记得`Drop`相关页，因为这是在B+树中，**从下往上的递归调用**（从上往下，使用了蟹行协议，把一些页面Drop了）

## P3 Query Execution

`cd build && make -j$(nproc) shell && ./bin/bustub-shell`

比较简单的一个project

有几个需要注意的地方：

* 仔细读文档：比如，不能修改`xxx_plan.h`，题目没有说`fit entirely in memory`，不能用数组存储中间结果。

* `Init`函数中，要记得对一些数组、变量初始化

  ![image-20240430104316300](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202404301043089.png)

* 在聚集函数中，出现空表情况，怎么处理？

  ```c++
  if (is_empty_) {
      // 这是一张空表
      // select id, min(score) from t1 group by id;  应该什么都不输出 （情况1）
      // select min(score) from t1 group by id;  输出integer-null  （情况2）
      auto value = aht_.GenerateInitialAggregateValue();
      std::vector<Value> values;
      values.reserve(GetOutputSchema().GetColumnCount());
      // 将聚集数组输出 min(score)
      values.insert(values.end(), value.aggregates_.begin(), value.aggregates_.end());
  
      // 如果是情况1，返回false
      if (values.size() != GetOutputSchema().GetColumnCount()) {
        return false;
      }
  
      // 如果是情况2，返回true
      *tuple = Tuple(values, &GetOutputSchema());
      is_empty_ = false;
      return true;
    }
  ```

* `Top-N`算法

  ```c++
  auto f = [](int a, int b){return a < b;};
  std::priority_queue<int, std::vector<int>, decltype(f)> q(f);
  ```

  * 当比较函数返回`true`，左边的变量后输出，右边的变量先输出（堆顶）
  * 上面这个函数，如果返回`true`，`b`会在堆顶，那么这是一个大根堆

  ```c++
  int findKthLargest(vector<int>& nums, int k) {
      priority_queue<int,vector<int>,greater<>>sq;
      for(auto &x:nums)  //让sq里面始终保持k个元素
      {
          if(sq.size()<k)
          {
              sq.push(x);
          }
          else if(sq.top()<x){
              sq.pop();
              sq.push(x);
          }
      }
      return sq.top();
  }
  ```

  * 上面这份代码是找第k大的数
  * 整个堆存放的是前k大的数，并且是一个小根堆
  * 理解上述代码，编写`Top-N`算法就简单多了。

* `Hash-Join`要处理碰撞

  ```c++
  for (std::size_t i = 0; i < plan_->LeftJoinKeyExpressions().size(); i++) {
      auto expr_left = plan_->LeftJoinKeyExpressions()[i];
      auto expr_right = plan_->RightJoinKeyExpressions()[i];
      auto vleft = expr_left->Evaluate(&left_tuple_, left_child_->GetOutputSchema());
      auto vright = expr_right->Evaluate(&right_tuple, right_child_->GetOutputSchema());
      if (vleft.CompareEquals(vright) == CmpBool::CmpTrue || (vleft.IsNull() && vright.IsNull())) {
          continue;
      }
      fg = false;
      break;
  }
  ```


## P4 Concurrency Control

### Task #1 - Lock Manager

* hierarchical table-level and tuple-level intention locks
* three isolation levels: `READ_UNCOMMITED`, `READ_COMMITTED`, and `REPEATABLE_READ`
* A failed lock attempt (such as for a deadlock) -> return false
* Any invalid lock operation should lead to an ABORTED transaction state (implicit abort) and throw an exception.

* you are familiar with the `Transaction` class's API and member variables
* read through `[LOCK_NOTE]`, `[UNLOCK_NOTE]`, and the LM's functions' specifications (in `lock_manager.h`).
* 在确保锁管理器能正常实施之后，再编写处理死锁的代码
* Take a look at the `LockRequestQueue` class in `lock_manager.h`. 判断哪些事务在等待锁
* 升级锁也要对LockRequestQueue进行修改
* We recommend using `std::condition_variable `provided as part of `LockRequestQueue`，当锁可用时，去通知事务
* using `*_lock_set_` ：记录事务获取的锁
*  `TransactionManager::Abort` 阅读这个函数，了解当abort时，锁管理器是怎么执行的
