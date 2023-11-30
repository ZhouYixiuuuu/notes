# vm.c 阅读源码

## walk

用**软件来模拟硬件**MMU查找页表的过程，返回以pagetable为根页表，经过多级索引之后va这个虚拟地址所对应的**页表项的地址**，如果alloc != 0，则在需要时创建新的页表页，反之则不用。

```c
pte_t *
walk(pagetable_t pagetable, uint64 va, int alloc)
{
  if (va >= MAXVA)
    panic("walk");

  // 这里只走了两层，最后pagetable是叶子页表
  for (int level = 2; level > 0; level--)
  {
    //pagetable 其实可以理解为一个数组
    //pagetable[PX(level, va)] 取出了其中的值
    //&pagetable[PX(level, va)] 取出它的地址
    //用pte_t * 接收它的地址
    //pte 是 地址
    //*pte 是 具体的值 也就是 pagetable[PX(level, va)]
    pte_t *pte = &pagetable[PX(level, va)];
    if (*pte & PTE_V)
    {
      // 这行代码从PTE中提取出物理地址，直接赋值给pagetable指针(而它是一个虚拟地址)
      // 这样赋值合理吗？只有在虚拟地址==物理地址时合理，即直接映射。
      pagetable = (pagetable_t)PTE2PA(*pte);
    }
    else
    {
      if (!alloc || (pagetable = (pde_t *)kalloc()) == 0)
        return 0;
      memset(pagetable, 0, PGSIZE);
      *pte = PA2PTE(pagetable) | PTE_V;
    }
  }
  return &pagetable[PX(0, va)];
}
```

## mappages

这个函数是用来装载一个新的映射

```c
// PGROUNDUP(sz)：sz大小的内存至少使用多少页才可以存下，返回的是下一个未使用页的地址
// PGROUNDDOWN(a)：地址a所在页面是多少号页面，拉回所在页面开始地址
#define PGROUNDUP(sz)  (((sz)+PGSIZE-1) & ~(PGSIZE-1))
#define PGROUNDDOWN(a) (((a)) & ~(PGSIZE-1))
```

给你一个虚拟地址，和一个物理地址，把它们两拼在一起，放在页表里面

```c
int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  // a存储的是当前虚拟地址对应的页
  // last存放的是最后一个应设置的页
  // 当 a==last时，表示a已经设置完了所有页，完成了所有任务
  uint64 a, last;
  pte_t *pte;

  // a,last向下取整到页面开始位置，设置last相当于提前设置好了终点页
  a = PGROUNDDOWN(va);
  last = PGROUNDDOWN(va + size - 1);
  for (;;)
  {
    if ((pte = walk(pagetable, a, 1)) == 0)
      return -1;
    if (*pte & PTE_V)
      panic("remap");
    *pte = PA2PTE(pa) | perm | PTE_V;
    if (a == last)
      break;
    a += PGSIZE;
    pa += PGSIZE;
  }
  return 0;
}
```

## kvminit/kvminithart

```c
// 将trampline页面映射到内核虚拟地址空间的最高一个页面
// TRAMPOLINE的定义如下，就是最高虚拟地址减去一个页面大小
// #define TRAMPOLINE (MAXVA - PGSIZE)
kvmmap(TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
```

在kvminit中其实是还没有分页机制的，在kvminithart中才开始有了分页机制

```c
void kvminithart()
{
  w_satp(MAKE_SATP(kernel_pagetable));
  sfence_vma();
}
```

自此之后虚拟地址**就要经过MMU的翻译**才可以转化为物理地址了

所以说前面的walk是模拟MMU来着，还没有开始用MMU呢，在kvminithart之后才开始用MMU

## walkaddr

专门用来查找用户页表中特定虚拟地址va所对应的物理地址。

**1.它只用来查找用户页表
2.返回的是物理地址，而非像walk函数那样只返回最终层的PTE的指针**

## freewalk

专门用来回收页表页的内存的

在调用这个函数时应该保证**叶子级别页表的映射关系全部解除并释放**(这将会由后面的uvmunmap函数负责)

![image-20231130151829589](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202311301518205.png)

```c
void freewalk(pagetable_t pagetable)
{
  // there are 2^9 = 512 PTEs in a page table.
  for (int i = 0; i < 512; i++)
  {
    pte_t pte = pagetable[i];
    //通过标志位判断是否到达了叶子页表
    //如果有效位为1，并且读/写/可执行位都是0
    //说明这不是叶子页表
    if ((pte & PTE_V) && (pte & (PTE_R | PTE_W | PTE_X)) == 0)
    {
      // this PTE points to a lower-level page table.
      uint64 child = PTE2PA(pte);
      freewalk((pagetable_t)child);
      pagetable[i] = 0;
    }
    else if (pte & PTE_V)
    {
      //表示这是一个叶级PTE，且未经释放，这不符合本函数调用条件，会陷入一个panic
      panic("freewalk: leaf");
    }
  }
  //最后释放页表本身占用的内存
  kfree((void *)pagetable);
}
```

