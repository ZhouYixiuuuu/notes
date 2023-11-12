# java IO流

## File的使用

* `File file = new File("F:\\软导作业");`可以既是一个文件也可以是一个文件夹
* `File[] files = file.listFiles();`罗列其中的所有文件和文件夹
* `isFile()`判断是否是个文件，`getName().endsWith("")`判断文件的后缀是否是某个字符串
* 遍历所有文件及其子文件夹，使用递归

## IO流

> 字节流，分为InputStream, OutputStream,具体对象为FileInputStream, FileOutputStream

`FileOutputStream fos = new FileOutputStream("src/a.txt");`

* 参数是字符串或者File对象都可以的
* 如果文件不存在会创造一个新文件，前提是父级文件夹存在
* 如果文件已经存在会清空文件
* **续写**`FileOutputStream fos = new FileOutputStream("src/a.txt", true);`

`fos.write("abs456".getBytes());`

* 一个字节一个字节的写
* **换行**`fos.write("\r\n".getBytes());`

`fos.close();`

* 每次使用完都要**释放资源**

`FileInputStream file = new FileInputStream("src/a.txt");`

* 不存在，会报错

`System.out.println((char)file.read());`

* 一次读一个字节，读到文件末尾，**返回-1**
* 记得定义**第三方变量**，因为一次读一个字节

`file.close();`

## 文件拷贝应用

```java
FileInputStream fis = new FileInputStream("F:/软导作业/aaa");
FileOutputStream fos = new FileOutputStream("F:/test");

int b;
while ((b = fis.read()) != -1)
{
    fos.write(b);
}
fos.close();
fis.close();
```

```java
FileInputStream fis = new FileInputStream("F:/软导作业/aaa");
FileOutputStream fos = new FileOutputStream("F:/test");
byte[] bytes = new byte[2];
//一次读取多个字节数据，具体多大，看数组的长度，建议数组长度都开成1024的整数倍
int len = fis.read(bytes);
//写的时候, 因为数组不一定填满了，所以要这样写
fos.write(bytes, 0, len);
fos.close();
fis.close();
```

