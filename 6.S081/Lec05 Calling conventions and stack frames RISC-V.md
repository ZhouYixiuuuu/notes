# Lec05 Calling conventions and stack frames RISC-V

## RISC-V and x86

* 一个RISC-V处理器，意味着这个处理器能够理解RISC-V指令集。所以任意一个处理器都有一个关联的ISA（Instruction Sets Architecture），每一条指令都有一个对应的二进制编码或者一个Opcode。当处理器在运行时，如果看见了这些编码，那么处理器就知道该做什么样的操作。
* 因为不同的处理器指令集不一样，**不同处理器对应的汇编语言必然不一样**
* 大多数现代计算机都运行在**x86和x86-64处理器**上。
* RISC-V中的**RISC是精简指令集**（Reduced Instruction Set Computer）的意思，而x86通常被称为**CISC，复杂指令集**（Complex Instruction Set Computer）。
* RISC指令数量更少，指令也更加简单，RISC-V的指令趋向于完成更简单的工作，相应的也消耗更少的CPU执行时间。相比x86来说，RISC另一件有意思的事情是**它是开源的**。这是市场上唯一的一款开源指令集，这意味着任何人都可以为RISC-V开发主板。
* 由于Intel的指令集是在是太大了，精简指令集的使用越来越多。Intel的指令集之所以这么大，是因为Intel对于**向后兼容**非常看重。所以一个现代的Intel处理器还可以运行30/40年前的指令。Intel并没有下线任何指令。

## RISC-V寄存器

![image-20231203195758858](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312031958462.png)

* **汇编代码并不是在内存上执行，而是在寄存器上执行**，也就是说，当我们在做add，sub时，我们是对寄存器进行操作。

  ```assembly
  add a0, a0, t0   #将a0寄存器上的值与t0寄存器上的值相加放入a0寄存器
  ```

* **ABI名字**：写汇编代码的时候使用的也是ABI名字。

* a0到a7寄存器是用来作为函数的参数。**如果一个函数有超过8个参数**，我们就需要用内存了。

* Caller Saved寄存器在函数调用的时候不会保存

  Callee Saved寄存器在函数调用的时候会保存

  举个例子：函数a调用函数b，任何被函数a使用过的Caller Saved寄存器，函数b可能重写这些寄存器。例如ra寄存器（return address，函数的返回地址），调用函数b时，ra寄存器会被重写。

* **所有的寄存器都是64bit**

## Stack

[参考]: https://textbook.cs161.org/memory-safety/x86.html

* `ebp` （frame pointer）：指向栈的底端，在RISC-V中，叫做`fp`
* `esp`（Stack pointer）：指向栈的顶端，在RISC-V中，叫做`sp`
* `eip`（instruction pointer）：指向当前函数的指令，在RISC-V中，叫做`PC`

![image-20231203200745104](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032007942.png)

```c
int main()
{
    foo();
    return 0;
}
void foo(int a, int b)
{
    int c = a + b;
}
```

1. 将main函数的参数入栈。这里需要注意：RISC-V会将8个参数存在寄存器，如果函数的参数多于8个，额外的参数会出现在Stack中。而x86直接将参数存在栈中。

![image-20231203201525799](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032015370.png)

2. 将旧的eip入栈，因为调用完foo函数，我们还要继续执行main函数，所以将main函数的eip入栈。

![image-20231203201651722](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032016005.png)

3. 移动eip，将eip移动到foo函数那里去

![image-20231203201751207](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032017021.png)

4. 将旧的ebp入栈，因为在调用foo函数时，ebp和esp的值都会改变，所以我们要先将ebp的值存起来。

![image-20231203201924027](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032019892.png)

5. 移动ebp到新的位置，就是现在esp所在的位置

![image-20231203202041238](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032020445.png)

6. 移动esp，为foo函数开辟一篇新的空间。

![image-20231203202115946](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032021278.png)

7. 执行函数
8. 移动esp，把空间还回来。

![image-20231203202216262](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032022550.png)

9. 将old ebp重新放入ebp

![image-20231203202410634](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032024363.png)

10. 更新eip

![image-20231203202448712](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312032024876.png)

11. 将main函数执行完，

 *function prologue*：（Steps 4-6）调用一个函数前，必须出现在`call foo`前面的那一段汇编代码--->保存ebp，将ebp和esp移动到新的位置去

*function epilogue*：（Steps 8-10）调用完一个函数，必须出现的一段汇编代码--->释放空间，将ebp，esp，eip归位

## Struct

当我们创建这样一个struct时，内存中相应的字段会彼此相邻。你可以认为struct像是一个数组，但是里面的不同字段的类型可以不一样。