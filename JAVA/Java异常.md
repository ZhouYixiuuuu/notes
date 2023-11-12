# Java异常

## 错误和异常

区别：灾难性致命的错误，JVM一般会选择终止线程，Exception通常是可以被程序处理的。

## 异常的常用命令

`try`监控区域

`catch`捕获异常

`finally`处理善后工作

`throw`主动地抛出异常

`throws`在方法上抛出异常

## 异常捕捉机制

![异常捕捉机制](F:\屏幕截图 2022-10-16 144831.png)

`e.printStackTrace()`可以查看异常抛出的过程

### 自定义异常

`extends Exception`