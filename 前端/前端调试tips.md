# 前端调试tips

##  console

多变量输出：`console.log({name, age, sex})`

输出错误信息：`console.error`

计算代码段执行时间：`console.time()` + `console.timeEnd()`

判断空值或判断false逻辑：`console.assert()`,第一项是`null`,`undefined`,`false`,0,会输出第二项。

```
console.assert(v, 'v is empty')
> Assertion failed: v is empty
```

输出dom节点：`console.dir()`

将数组或对象按表格输出：`console.table()`

## 断点

`source`源代码页面直接点击对应行数

vscode 在终端面板选择调试终端`javasccript Debug Terminal`

