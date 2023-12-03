# gdb的使用

gdb配置

```shell
echo "add-auto-load-safe-path YOUR_PATH/xv6-labs-2020/.gdbinit " >> ~/.gdbinit
```

Open qemu under gdb mode:
`make qemu-gdb`
Attach gdb to qemu in a second terminal:
`gdb-multiarch`

如果以单核方式启动，则使用`make CPUS=1 qemu-gdb`

当你不知道某个命令怎么使用时，运行`help <命令名称>`来获取帮助

## 单步调试

- `step`一次运行一行代码。当有函数调用时，它将步进到被调用的对象函数。
- `next`也是一次运行一行代码。但当有函数调用时，它不会进入该函数。
- `stepi`和`nexti`对于汇编指令是单步调试。

## 运行调试

- `continue`运行代码，直到遇到断点或使用`<Ctrl-c>`中断它
- `finish`运行代码，直到当前函数返回
- `advance <location>`运行代码，直到指令指针到达指定位置

## 断点

- `break <location>`在指定的位置设置断点。 位置可以是内存地址(`*0x7c00`)或名称(`monbacktrace`，`monitor.c:71`)
- 如需修改断点请使用`delete`，`disable`，`enable`

## 条件断点

- `break <location> if <condition>`在指定位置设置断点，但仅在满足条件时中断。
- `cond <number> <condition>`在现有断点上添加条件。

## 检查命令

* `print`计算一个C表达式并将结果以合适的类型打印。

- 使用`p *((struct elfhdr *) 0x10000)`的输出比`x/13x 0x10000`的输出好得多

## 其他检查命令

- `info registers`打印每个寄存器的值
- `info frame`打印当前栈帧
- `info mem`：打印页表
- `list <location>`在指定位置打印函数的源代码
- `backtrace`或许对于你的lab1中的工作很有用处

## 布局

`layout asm`：查看汇编

`layout reg`：查看寄存器

`info reg`：查看寄存器

`layout split` 

`tui enable`可以打开源代码展示窗口



