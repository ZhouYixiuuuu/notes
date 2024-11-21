# Lec08 Page faults

内核需要什么信息才能响应page fault？

1. 出错的虚拟地址（**存在STVAL寄存器中**）
2. 出错的原因（**存在SCAUSE寄存器中**，这个寄存器保存了trap机制中进入内核态的原因）
3. 触发page fault的指令的位置（修复完page fault，可以继续执行）（**存在SEPC寄存器中**）



sbrk->eager alloration

申请多于自己所需的内存，进程的内容会消耗很多

## lazy allocation

eager allocation：应用程序很难预测自己需要多少内存，所以会申请多于自己需要的内存，有些内存就会浪费。

lazy allocation：分配了虚拟内存，但没有对应的物理内存。应用程序使用到了新申请的那部分内存，这时会触发page fault。sbrk系统调基本上不做任何事情，唯一需要做的事情就是提升*p->sz*，将*p->sz*增加n，其中n是需要新分配的内存page数量。

此时page fault的响应：

- 在page fault handler中，通过kalloc函数分配一个内存page；
- 初始化这个page内容为0；
- 将这个内存page映射到user page table中；
- 最后重新执行指令。

如果使用的地址低于*p->sz*，那么这是一个用户空间的有效地址。如果大于*p->sz*，对应的就是一个程序错误。

修改代码：

1. `sbrk`不分配内存

![image-20231218105826079](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312181058046.png)

2. 在`usertrap`函数中，为`lazy allocation`进行定制化服务（分配一个新页面，初始化页面，在用户页表进行映射）

![image-20231218105932318](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312181059188.png)

3. 在`uvmunmap`函数中，释放用户页表，`lazy allocation`存在一些没有物理内存对应的mapping，所以

   ![image-20231218110202335](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312181102331.png)

   这里会报错，改成`continue`就好了

## Zero Fill On Demand

当编译器在生成二进制文件时，编译器会填入这三个区域。

- text区域是程序的指令，
- data区域存放的是初始化了的全局变量，
- BSS包含了未被初始化或者初始化为0的全局变量。

BSS区域里面可能有好几个page，里面初始化都是0，那么我只需要分配一个page，这个page初始化为0，让BSS中全0的page都映射到这个物理page中。

![image-20231218112324127](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312181123622.png)

所有的映射，不能对这个物理page进行**写操作**，不然就会修改它的全0。

当应用程序想修改这个page时，会得到一个page fault，那就重新建一个page，更新这个page的映射关系。

![image-20231218112648736](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312181126206.png)

这里也有个弊端：就是有一次update操作，就执行一次page fault，从用户空间到内核空间，会付出一定的代价。

## copy-on-write fork

原本的`fork`：`fork`创建一个子进程，会拷贝父进程的物理内存，但是一旦调用了exec函数，又会释放这些page，并重新分配page给新的指令。

`COW fork`：`fork`时，与其复制，不如共享。将子进程的PTE指向父进程的物理page。同理，为了保证进程的隔离性，这里的PTE的标志位都设置成只读的。

在某个时间点，我们需要更改内存的内容，就会得到page fault

![image-20231218113653291](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312181136051.png)

**内核需要能辨别出`COW fork`的场景**，不是每次page fault都会指向该操作，这里会使用PTE的RSW标志位（Reserved for supervisor software），标志当前是否是一个`COW fork`的page。

**释放父进程的时候，要释放对应的物理内存吗？**

答案是，如果它有子进程的话，不释放对应的物理内存。我们需要对每个物理内存page的引用进行计数，当我们释放一个虚拟page，引用数-1，当引用数为0，就释放当前物理page。

## demand paging

在虚拟地址空间中，我们为text和data分配好地址段，但是相应的PTE并不对应任何物理内存page。对于这些PTE，我们只需要将valid bit位设置为0即可。

跟上面的Zero FIll On Demand很像，连指令都不分配物理地址，用到再分配。

我们需要在某个地方记录虚拟page对应的程序文件，当触发page fault的时候，再去对应的程序文件中读取数据到内存，最后再执行指令。

**物理内存耗尽了怎么办？**

将部分内存page的内存写回文件系统，再撤回page，可以用这个空闲的page，分配给刚刚那个page fault handler。

**选哪个page进行撤回？**

* `dirty page`：曾经被写过
* `non-dirty page`：只被读过，没被写过

选`non-dirty page`进行撤回：如果选`dirty page`进行撤回，后续`dirty page`再被修改，你就要写两次，一次写在内存，一次写在磁盘。

**`LRU`算法**：找到一定时间内没有被访问过的page，将其撤回。使用PTE的`Access bit`标志位，记录该页面是否被访问过，并且**定期刷新**。使用clock algorithm。

## memory mapped files

`mmap(va, len, prot, flags, fd, off)`：从文件描述符对应的文件偏移量的位置开始，映射长度为len的内容到虚拟内存地址va，同时加上一些flags

`unmap(va, len)`：将`dirty block`写回文件中

使用VMA结构体（Virtual Memory Area）记录当前PTE属于这个文件描述符，还有偏移量

