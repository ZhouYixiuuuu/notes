# Java期末复习

## java基础

### 定义和使用基本数据类型

byte, short, int, long, float, double, boolean, char(八种数据类型)

### 定义常量和变量

常量使用`final`修饰

### 输入和输出

输入

```java
Scanner sc = new Scanner(System.in);
String str = sc.next();  // 读入下一个字符串
```

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String str = br.readLine();
        System.out.println(str);
    }
}
```

输出

```java
System.out.println(123);  // 输出整数 + 换行
System.out.printf("%04d %.2f\n", 4, 123.456D);  // 格式化输出，float与double都用%f输出
//没有%lf，注意一下！！！
```

```java
import java.io.BufferedWriter;
import java.io.OutputStreamWriter;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        bw.write("Hello World\n");
        bw.flush();  // 需要手动刷新缓冲区
    }
}
```



### 数组的定义和遍历

数组的定义

`int[] a = new int[10];`

数组的初始化

`int[] b = new int[3];       // 含有3个元素的数组，元素的值均为0`

数组的遍历

`for (int x: row)`

常用API

长度：`length`**注意不加小括号**

排序：`Arrays.sort()`

转化成字符串：`Arrays.toString()`

多维数组转化成字符串：`Arrays.deepToString()`

判断两个数组是否相等 : `Arrays.equals(arr1, arr2)`

### 字符串

**只读变量, 不能修改**

```java
String a = "Hello ";
a += "World";  // 会构造一个新的字符串
```

格式化字符串: `String str = String.format("My age is %d", 18);  // 格式化字符串，类似于C++中的sprintf`

* 访问`String`中的字符：`str.charAt()`，只能读取，不能写入

* 长度：`length()`

* 判断相等：`equals() ` **注意不能直接用==**

* 比较字典序大小：`compareTo()` 负数表示小于，0表示相等，正数表示大于

* 子串：`substring(int beginIndex, int endIndex)`左闭右开

* 分隔字符串：`split(String regex)`

******

查找：`indexOf(char c), indexOf(String str), lastIndexOf(char c), lastIndexOf(String str)`找不到返回-1

判断是否以某个前缀开头：`startsWith()`

判断是否以某个后缀结尾：`endsWith()`

全部用小写字母：`toLowerCase()`

全部用大写字母：`toUpperCase()`

替换：`replace(char oldChar, char newChar)` , `replace(String oldRegex, String newRegex)`

将字符串转化为字符数组：`toCharArray()`

String转化为Double：	`Double.parseDouble(str)`

去掉首尾的空白字符: `trim()`

************

String不能被修改，如果打算修改字符串，可以使用`StringBuilder`和`StringBuffer`

`StringBuffer`线程安全，速度较慢；`StringBuilder`线程不安全，速度较快。

```java
StringBuilder sb = new StringBuilder("Hello ");  // 初始化
sb.append("World");  // 拼接字符串
System.out.println(sb);

for (int i = 0; i < sb.length(); i ++ ) {
    sb.setCharAt(i, (char)(sb.charAt(i) + 1));  // 读取和写入字符
}

System.out.println(sb);
```

`reverse()`：翻转字符串

### 函数

静态函数中只能调用静态函数和静态变量。

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        System.out.println(max(3, 4));
        int[][] a = new int[3][4];
        fill(a, 3);
        System.out.println(Arrays.deepToString(a));

        int[][] b = getArray2d(2, 3, 5);
        System.out.println(Arrays.deepToString(b));
    }

    private static int max(int a, int b) {
        if (a > b) return a;
        return b;
    }

    private static void fill(int[][] a, int val) {
        for (int i = 0; i < a.length; i ++ )
            for (int j = 0; j < a[i].length; j ++ )
                a[i][j] = val;
    }

    private static int[][] getArray2d(int row, int col, int val) {
        int[][] a = new int[row][col];
        for (int i = 0; i < row; i ++ )
            for (int j = 0; j < col; j ++ )
                a[i][j] = val;
        return a;
    }
}
```



### 常用类库的使用

random的使用

```java
Random random = new Random()
random.nextInt(int n);  //生成一个介于[0, n)之间的数
```

Scanner是可以读文件的

```java
File file = new File("test.txt");
Scanner sc = new Scanner(file);
while (sc.hasNext())
{
    String temp = sc.nextLine();
    System.out.println(temp);
}
```



## 面向对象编程

### 类

定义类时可以使用的修饰符有：public, abstract, final

### 继承、多态

### 接口

```c++
interface Role {
    public void greet();
    public void move();
    public int getSpeed();
}
```

### 包

## 异常处理

*  catch 一定要和try在一起

* throw：在函数内抛出一个异常
* throws：在函数定义时抛出异常

```c++
private static void foo() throws IOException, NoSuchFieldException 
{
    Scanner sc = new Scanner(System.in);
    int x = sc.nextInt();
    if (x == 1)
        throw new IOException("找不到文件！！！");
    else
        throw new NoSuchFieldException("自定义异常");
}
```



## 集合类

### List

感觉有点像vector..

* 变长数组

  `List<Integer> list = new ArrayList<>();`

* 双链表

  `List<Integer> list = new LinkedList<>();`

函数:

* `add()` 添加一个元素
* `clear()` 清空
* `size()` 返回长度
* `isEmpty()` 是否为空
* `get(i)` 获取第i个元素
* `set(i, val)` 将第i个元素设置为val

遍历:

```java
for (Integer element: list) {
    System.out.println(element);
}
```

### Deque

可以当栈用

虽然它是双端队列

`Deque<Integer> stack = new ArrayDeque<>()`

`stack.push(value)` 入栈

`stack.pop()` 出栈

`stack.peek()`  返回栈顶元素

`stack.size()` 返回栈的大小

`stack.isEmpty()`判断栈是否为空

### Queue

`Queue<Integer> q = new LinkedList<>()`;

优先队列 `Queue<Integer> q = new PriorityQueue<>()`;

默认是小根堆 大根堆写法是 `Queue<Integer> q = new PriorityQueue<>`

* `add()` 入队
* `remove()` 出队并返回队头
* `isEmpty()` 
* `size()`
* `peek()`
* `clear()`

### Set

`HashSet`类：不保证顺序

`TreeSet`类：保证顺序

定义

`Set<Integer> set = new TreeSet<>()`

### Map

`Map<String, String> map = new HashMap<>()`

`TreeMap<Integer, Integer> map = new TreeMap<>()`

遍历

```java
for(Map.Entry<Integer, String> entry : map.entrySet())
{
    System.out.println(entry.getKey());
    System.out.println(entry.getValue());
}
```



## 文件的输入和输出

`File file = new File("")`创建一个文件对象

`FileOutputStream out = new FileOutputStream(file)`文件输入和输出流

`out.write("")`

`out.close()`记得关闭

`file.createNewFile()`创建一个文件

`file.exists()`判断文件是否存在

## 数据库操作

`DriverManager`是负责加载数据库的驱动程序的

创建connection对象：`Connection con = DriverManager.getConnection("")`

向数据库发送SQL语句：`Statement st = con.createStatement()`

查询数据：`ResultSet res = st.executeQuery("")`

关闭连接：`con.closeConnection()`

## 多线程

## 网络通信

`InetAddress`类：获取ip地址，主机地址

`ServerSocket`类：表示服务器套接字，等待来自网络上的的“请求”，`accept()`等待客户机的连接
