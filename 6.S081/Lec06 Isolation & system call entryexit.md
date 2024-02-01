# Lec06 Isolation & system call entry/exit

https://blog.csdn.net/zzy980511/article/details/130255251

用户空间和内核空间的切换称为trap

以下三种情况会出现trap：

1. 程序执行系统调用
2. 异常（page fault、运算除以0之类的）
3. 设备中断（磁盘硬件完成读写请求之类的）

从用户态变成内核态，**硬件**也要从适合运行用户代码的状态，改变成适合运行内核代码的状态。

1. 32个用户寄存器要找个地方存起来
2. 修改mode标志位，user mode -> supervisor mode
3. pc也要找个地方保存，回来时才可以从中断的地方继续执行。pc原本指向用户代码，改成指向内核代码
4. SATP：用户页表->内核页表
5. 将堆栈寄存器指向内核的一个地址，才能执行内核的函数
6. 还有一些在内核态才能修改的寄存器，如SATP、STVEC、SEPC、SSRATCH等等都要修改。
   * SATP：指向页表的物理内存地址
   * STVEC：指向内核中处理trap指令的起始地址
   * SEPC：在trap的过程中保存程序计数器的值
   * SSRATCH

supervisor mode究竟有哪些权限呢？

1. 读写SATP寄存器，也就是page table的指针；STVEC，也就是处理trap的内核指令地址；SEPC，保存当发生trap时的程序计数器；SSCRATCH等等。在supervisor mode你可以读写这些寄存器，而用户代码不能做这样的操作。
2. 当PTE_U标志位为1的时候，表明用户代码可以使用这个页表；如果这个标志位为0，则只有supervisor mode可以使用这个页表。
3. supervisor mode中的代码并不能读写任意物理地址。只能通过页表访问内存。

## trap代码的执行流程

![image-20231204122144809](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312041221216.png)

### `ecall` -> `trampoline`

```assembly
.global write
write:
 # 将write的系统调用号写入a7寄存器
 li a7, SYS_write
 # 使用ecall触发一个陷阱
 ecall
 ret
```

`ecall`会做什么？主动触发一个用户态异常，进而导致一系列trap动作，它们都是由硬件自动完成的：

1. 修改mode：user mode -> supervisor mode
2. 将当前pc的值存入sepc寄存器中保存
3. 将stvec中保存的`trampoline`程序的入口地址放入pc中，准备进入
4. 将当前导致陷阱的原因记录在scause寄存器中。
5. 将当前模式保存在sstatus的SPP位，并清空sstatus中的SIE位来关闭中断，之前的SIE为保存在SPIE位。
6. 更新stval寄存器的值，使其指向出现异常的地址

下面进入trampoline中的uservec函数，在这个函数中，做了以下事情

1. 保存32个寄存器的值，每个进程的用户页表都有一个`trapframe`，我们将寄存器的值保存在这里，`sscratch`寄存器指向的就是进程`p->trapframe`在内存中的映射地址，也就是`TRAPFRAME`
2. 将sp指向内核栈
3. 将用户页表变成内核页表
4. 进入下一个要执行的函数`usertrap()`

```assembly
.globl uservec
uservec:    
	#
    # trap.c sets stvec to point here, so
    # traps from user space start here,
    # in supervisor mode, but with a
    # user page table.
    # 当前处于supervisor模式，但是页表还是用户页表
    #
    # 保存寄存器的值，需要一个空闲的通用寄存器来寻址
    # 所以我们对调sscratch和a0的寄存器的值，把a0空出来，用来寻址
    # swap a0 and sscratch
    # so that a0 is TRAPFRAME
    csrrw a0, sscratch, a0
    
    # save the user registers in TRAPFRAME
    # 偏移多少，可以在proc.h中看到
    sd ra, 40(a0)  # 将ra寄存器的值，放到a0地址偏移40的位置
    sd sp, 48(a0)
    sd gp, 56(a0)
    sd tp, 64(a0)
    ...... 这里把代码省略，剩下就是三十多个寄存器，一个一个拷贝过去
    
    # 再把a0存回来
    csrr t0, sscratch
    sd t0, 112(a0)
    
    # 设置内核栈指针
    # 在系统启动时，就为每个进程在全局内核页表分配一个内核栈了
    ld sp, 8(a0)
    
    # 将trapframe中存放的cpu的id号放入tp寄存器
    # hartid是RISC-V给每个CPU的一个ID号，内核态下必须保存在tp寄存器中
    ld tp, 32(a0)
    
    # 将p->trampframe->kernel_trap的地址放入t0寄存器，准备跳入
    # 这里就是usertrap()这个函数的地址，放入t0中
    ld t0, 16(a0)
    
    # 切换内核页表，注意清空快表
    ld t1, 0(a0)
    csrw satp, t1
    sfence.vma zero, zero
    
    # 现在进入内核页表了，a0已经没用了，因为内核页表没有trapframe了
    
    # 跳入usertrap()函数
    jr t0
```

### `usertrap(kernel/trap.c)`

usertrap就像是一个中转站，它**判断陷阱的原因**并**将陷阱分发到对应的处理句柄**(handler)。

```c
// 代码框架
// 1. 处理系统调用
if (r_scause() == 8) {
    ....
    syscall();
}
// 2. 处理设备中断
else if (interrupt) {
    devintr();
}
// 3. 处理程序错误
else{
    kill process;
}
```

```c
  //判断陷阱来源是否是用户模式
  if((r_sstatus() & SSTATUS_SPP) != 0)
    panic("usertrap: not from user mode");

  // send interrupts and exceptions to kerneltrap(),
  // since we're now in the kernel.
  //我们现在在内核模式下
  //要是我们现在也出现了中断 -> 就是内核中断
  //处理内核中断的代码不是trampoline，而是kernelvec代码
  //将其放入stvec寄存器中
  w_stvec((uint64)kernelvec);

  struct proc *p = myproc();
  
  // save user program counter.
  //因为这里可能会触发定时器中断，导致进程切换
  //sepc就会被重写
  //所有将sepc的值存在进程的trapframe中
  p->trapframe->epc = r_sepc();
```

```c
if(r_scause() == 8){
    // system call

    //如果进程被杀死，则直接退出
    if(p->killed)
        exit(-1);

    // sepc points to the ecall instruction,
    // but we want to return to the next instruction.
    // 我们希望执行完系统调用后，可以执行它的下一条指令
    p->trapframe->epc += 4;

    // an interrupt will change sstatus &c registers,
    // so don't enable until done with those registers.
    // 开中断
    intr_on();

    // 执行系统掉牙
    syscall();
}
```

### `syscall()`

```c
void
syscall(void)
{
  int num;
  struct proc *p = myproc();

  //取出系统调用号
  num = p->trapframe->a7;
  //判断系统调用号是否合法
  //合法则执行系统调用
  //将系统调用的结果存在trapframe的a0中
  //这个值会在userret汇编代码中恢复给用户态的a0
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    p->trapframe->a0 = syscalls[num]();
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}
```

### `usertrapret()`

执行完`syscall()`，回到`usertrap()`，执行`usertrapret()`函数

```c
void
usertrapret(void)
{
  struct proc *p = myproc();

  // we're about to switch the destination of traps from
  // kerneltrap() to usertrap(), so turn off interrupts until
  // we're back in user space, where usertrap() is correct.
  // 关中断(直到执行SRET之前，始终不响应中断)
  intr_off();

  // send syscalls, interrupts, and exceptions to trampoline.S
  //将stvec设置成trampoline中的uservec
  w_stvec(TRAMPOLINE + (uservec - trampoline));

  // set up trapframe values that uservec will need when
  // the process next re-enters the kernel.
  // 分别设置trapframe中的值，这些值下次中断的时候，还会被用到
  p->trapframe->kernel_satp = r_satp();         // kernel page table
  p->trapframe->kernel_sp = p->kstack + PGSIZE; // process's kernel stack
  p->trapframe->kernel_trap = (uint64)usertrap;
  p->trapframe->kernel_hartid = r_tp();         // hartid for cpuid()

  // set up the registers that trampoline.S's sret will use
  // to get to user space.
  
  // set S Previous Privilege mode to User.
  unsigned long x = r_sstatus();
  x &= ~SSTATUS_SPP; // clear SPP to 0 for user mode
  x |= SSTATUS_SPIE; // enable interrupts in user mode
  w_sstatus(x);

  // set S Exception Program Counter to the saved user pc.
  // 这里trapframe中的epc的值已经是下一条指令的值了
  // 将其赋给sepc寄存器
  w_sepc(p->trapframe->epc);

  // tell trampoline.S the user page table to switch to.
  // 准备切换页表，准备好要写入satp的值
  uint64 satp = MAKE_SATP(p->pagetable);

  // jump to trampoline.S at the top of memory, which 
  // switches to the user page table, restores user registers,
  // and switches to user mode with sret.
  // 获取userret这个函数的虚拟地址 fn
  // 将虚拟地址直接作为函数指针进行调用，并传入两个参数
  // 根据函数调用，a0寄存器的值现在是TRAPFRAME，a1寄存器的值是satp
  uint64 fn = TRAMPOLINE + (userret - trampoline);
  ((void (*)(uint64,uint64))fn)(TRAPFRAME, satp);
}
```

### `userret`

```assembly
# usertrapret() calls here.
# a0: TRAPFRAME, in user page table.
# a1: user page table, for satp.

# 先改个页表
csrw satp, a1
sfence.vma zero, zero

# 现在a0是TRAPFRAME，根据上一个usertrapret()可知
# 112(a0)就是trapframe中保存的a0，将其与sscratch互换
ld t0, 112(a0)
csrw sscratch, t0

# 从TRAPFRAME中恢复除了a0以外的所有寄存器
......

# 把a0和sscratch换回来
csrrw a0, sscratch, a0

# 回到用户模式
sret
```

`sret`做了什么？

1. 将`sepc`中的值放入`pc`（`sepc`中的值是下一条指令的位置了）
2. 更改成用户模式
3. 将权限模式设置为`sstatus`中的SPP位，这在`usertrapret`中也已经设置好了，0表示返回用户模式
4. 将`sstatus`中SIE位的值设置为SPIE，在`usertrapret`中SPIE被设置为1表示开启中断
5. 最后将`sstatus`中的SPP置为0，SPIE置为1

## Lab4 Traps

### RISC-V assembly

1. 在a0-a7中存放参数，13存在a2中

2. 编译器优化之后没有进行函数调用

3. 在0x630

4. 0x38

5. HE110 World
   x = 3, y = 1

   因此y=后面打印的结果取决于之前a2中保存的数据

### Backtrace

```c
void backtrace()
{
  //获取当前正在执行的函数的帧指针
  uint64 fp = r_fp();
  printf("backtrace:\n");
  //没到当前帧的最顶端，就一直往上走
  while (fp < PGROUNDUP(fp))
  {
    uint64 ra = *(uint64 *)(fp - 8);
    fp = *(uint64 *)(fp - 16);
    printf("%p\n", ra);
  }
}
```

![image-20231208162238093](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312081622829.png)

### Alarm

每当程序消耗了CPU时间达到n个“滴答”，内核应当使应用程序函数`fn`被调用。

在XV6中，一个滴答是一段相当任意的时间单元，取决于硬件计时器生成中断的频率。

添加两个系统调用

1.  `sigalarm(interval, handler)` 主要起到一个初始化的作用，就是将滴答的个数和需要被调用的函数，放到进程结构体里面去
2. **每一个滴答声，硬件时钟就会强制一个中断，这个中断在kernel/trap.c\中的`usertrap()`中处理。**
3. 那么我们的思路就是：在进程结构体里面设置一个cnt记录距离上次调用`handler`过了多少个滴答了，如果过了`sigalarm`函数规定的滴答数，就调用`handler`，并把cnt置为0
4. 在调用完`handler`之后，我们还要继续执行原函数，所以在执行`handler`之前，需要把此刻的`p->trapframe`存下来，调用完`handler`再变回去。
5. 变回去就是在`sigreturn`函数中进行
6. 防止对处理程序的重复调用——如果处理程序还没有返回，内核就不应该再次调用它。设置一个`Inhandler`变量，判断发生中断时，是否正处于`handler`函数中，防止重复调用`handler`函数。

```c
uint64 sys_sigalarm(void)
{
  if (argint(0, &myproc()->ticks) < 0) return -1;
  if (argaddr(1, &myproc()->handler) < 0) return -1;
  //printf("%p\n", myproc()->handler);
  return 0;
}
```

```c
  // give up the CPU if this is a timer interrupt.
  if (which_dev == 2)
  {
    if (p->ticks != 0 && p->Inhandler == 0)
    {
      p->ticks_cnt++;
      if (p->ticks_cnt == p->ticks)
      {
        // 调用函数handler
        memmove(p->alermframe, p->trapframe, sizeof(struct trapframe));
        p->trapframe->epc = p->handler;
        p->Inhandler = 1;
        p->ticks_cnt = 0;
      }
    }
    yield();
  }
```

```c
uint64 sys_sigreturn(void)
{
  struct proc * p = myproc();
  p->Inhandler = 0;
  memmove(p->trapframe, p->alermframe, sizeof(struct trapframe));
  return 0;
}
```

![image-20231208163528848](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312081635516.png)

需要注意的几个点：

1. 系统调用的参数在哪里获得？在寄存器啊，不要弄错了。

2. 学会使用`memmove`函数，进行内存的复制

3. ```c
   memmove(p->alermframe, p->trapframe, sizeof(struct trapframe));
   p->trapframe->epc = p->handler;
   ```

   这两行的先后顺序不能写错！

![image-20231208161709433](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312081617182.png)