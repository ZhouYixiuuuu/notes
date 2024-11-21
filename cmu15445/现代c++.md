# 现代c++

## 编译

* 常见的编译器有：clang和g++

```she
g++ -std=c++17 main.cpp -o main
```

* 指明编译器
* 指明c++版本
* 指明源文件
* `-o`：This means that you’re going to give a specific name to your executable

## 函数重载

```c++
double half(double x) {
	return x / 2;
}

int half(int x, int divisor = 2)
{
	return x / divisor;
}

int main()
{
	cout << half(3.0) << endl << half(3, 3);
	return 0;
}
```

## pair

STL 有自己的结构体

```c++
std::pair<bool, Student>
std::make_pair(false, blank)
```

## 初始化

* Direct initialization
* Uniform initialization

```c++
int numOne(12.3);  //Direct initialization  12
int numTwo{12.3};  //Uniform initialization 会报错
```

Uniform initialization

1. 安全的，不会发生强制类型转换

2. 适用性广

   ```c++
   std::map<std::string, int> ages{
       {"Alice", 25},
       {"Bob", 30}
   };
   
   struct Student {
       string name;
       string state;
   };
   
   Student s{"Haven", "AR"};
   ```

### 结构化绑定

Structured Binding

更简洁

```c++
std::tuple<std::string, std::string, std::string> getClassInfo() {
	std::string className = "CS106L";
	std::string buildingName = "Turing Auditorium";
    std::string language = "C++";
    return {className, buildingName, language};
}

auto [className, buildingName, language] = getClassInfo();

/*
auto classInfo = getClassInfo();
std::string className = std::get<0>(classInfo);
std::string buildingName = std::get<1>(classInfo);
std::string language = std::get<2>(classInfo);
*/
```

增加代码的可读性

```c++
遍历map
for (const auto& [key, val] : mymap)
{
    std::cout << key << ": " << val << endl;
}
```

下面这个代码为什么没有修改nums数组的值？

```c++
void shift(std::vector<std::pair<int, int>> & nums)
{
    for (auto [num1, num2] : nums) {
        num1 ++;
        num2 ++;
    }
}
```

**num1和num2不是引用。**下面这样就是正确的。

```c++
void shift(std::vector<std::pair<int, int>> & nums)
{
    for (auto& [num1, num2] : nums) {
        num1 ++;
        num2 ++;
    }
}
```

## const

```c++
const std::vector<int> vec{1, 2, 3};
std::vector<int>& const_ref{vec};  //会报错
const std::vector<int>& const_ref{vec};
```

* **You can’t declare a non-const reference to a const variable**

**去除const**

`const_cast<>()`

现在有一个类

```c++
class Student {
Private:
    /// An example of type aliasing
    using String = std::string;
    String name;
    String state;
    int age;
public:
    Student(String name, String state, int age);
    /// Added a ‘setter’ function
    void setName(String name);
    String getName();
    String getState();
    int getAge();
}
```

有一个函数

```c++
std::string stringify(const Student& s){
    return s.getName() + " is " + std::to_string(s.getAge()) +
    " years old." ;
}
```

为什么会报错？

* 因为函数传入的参数是个const类型，我们是不能修改参数s
* 但是编译器不知道`s.getName()`和`s.getAge()`是否会修改参数s
* 成员函数如果不特别声明，总是可以访问和修改成员变量的。

修改成下面这样

```c++
//.h file
String getName() const;
int getAge() const;
```

```c++
//.cpp file
std::string Student::getName() const {
	return this->name;
}

int Student::getAge() const {
	return this->age;
}
```

* **const 对象只能和 const接口交互**
* **const 接口 不能修改成员变量**

```c++
int& findItem(int value) {
    for (auto& elem: arr) {
    if (elem == value) return elem;
    }
    throw std::out_of_range(“value not found”)
}
const int& findItem(int value) const {
    /// one-liners ftw :)
    return const_cast<IntArray&>(*this).findItem(value)
}
```

* 很抽象的一个函数，当某个const成员函数，要返回一个引用时，返回的引用也要是const

## 引用

指针的语法糖

没有实际的物理内存，只是取了一个别名(aliases)

函数通过**传递引用**才能实际修改参数值。

```c++
int a = 5;
int& ref = a;
a = 2;
std::cout << ref << std::endl;  //2

int f(int& a){}  //传递引用
```



## Streams

![image-20240317161132333](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403171611973.png)

### std::stringstream

* `>>`操作符，按**空格**读取

* `getline()` 按行读取

   `istream& getline(istream& is, string& str, char delim)`

  * input stream is
  * store in str
  * **截止符 delim（默认是 '\n'）**
  * `getline()`的读取的字符串是不包括delim的

* 如果将`std::cin`和`getline()`一起使用，**一定一定要记得在`getline()`前面加多一个`getchar()`，读取多的换行。**

  ![image-20240317172807543](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403171728794.png)

***Don’t use getline() and std::cin() together***

### Output Streams

![image-20240317165913837](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403171659713.png)

* `std::cout << std::flush;` 可以清空缓冲区
*  `std::endl` 输出换行后，也会清空缓冲区

![image-20240317170236084](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403171702762.png)

**Output file streams**

* `std::ofstream`

  ```c++
  std::ofstream ofs("hello.txt");
  if (ofs.is_open())
  {
      std::cout << "ofs open" << std::endl;
      ofs << "Hello CS106L" << '\n';
  }
  ofs.close();
  ofs << "this will not get written";
  ofs.open("hello.txt");
  ofs << "this will though! It's open again";
  ```

  ```c++
  ofs.open("hello.txt", std::ios::app);
  ofs << "append";
  ```

**Input file streams**

```c++
std::ifstream ifs("hello.txt");
std::string line;
std::getline(ifs, line);
std::cout << line;
```

## vector

* `erase()`函数：删除单个元素，迭代器指向被删除元素的位置，被删除元素之后的所有元素都向前移动以为。**迭代器实际上指向了原来被删除元素的下一个元素。**

  使用for循环时，这里记得--

## containers

**Sequence containers:**

 ○ Arrays, vectors, deques, lists 

 **Associative containers:** 

○ Sets and maps 

○ Unordered vs. ordered

### Container Adapter

基于现有容器类型（如vector、list、deque等）的封装，提供一些新的操作接口，使得原有容器类型能够更加灵活地应对不同的场景需求。

常见的容器适配器有：stack、queue、priority_queue

* stack  --> deque
* queue  --> deque
* priority_queue  --> vector

## Interators

**Iterators are a type of pointer!**

![image-20240318111533272](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403181115454.png)

```c++
std::set<int> myset{1, 2, 3, 4};

// define a iterator like this: 
// container_class_name::iterator iterator_name;
std::set<int>::iterator iter = myset.begin();
cout << *iter << endl; // 1
```

**将iterator变成指针：`auto ptr = &*it`**

### Types

![image-20240318164920369](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403181649106.png)

1. Input Iterator：能够读取值 `auto elem = *it;`
2. Output Iterator：能够写入值（修改值）`*elem = value;`
3. Forward iterators：单向遍历
4. Bidirectional iterators：双向遍历 `++it`、`--it`
5. Random-access iterators：随机访问（**常数** 时间内） `it += 5` 、`vec[1]`

![image-20240318165237719](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403181652087.png)

**question：**

* 对于 `map<int, int> m;` ，我们使用 `sort(m.begin(), m.end());` 可以吗？

  sort 的实现本质上是快速排序（实际上不完全是），而快速排序本身是需要 **随机访问** 容器中的元素的，map 本身并不能够满足上述要求。

  

## 头文件

`#include "xxxx.h"`

可以在xxx.h里面声明一些函数

`#pragma once` 预处理器命令，阻止单个头文件被多次包含

```c++
//A.cpp
#include "log.h"
#include "b.h"

//log.h
#include "b.h"

//如果log.h中没有#pragma once，b.h就会被重复复制
```

## static

在变量或者函数前加多一个static，它只会在它被声明的c++文件中被看见，例如在头文件中的变量加多一个static，其他文件引用这个头文件时，无法访问被添加static的函数或变量。（有点像类中的private）

如果没有设定为static，链接器会跨编译单元进行链接（全局作用域）

类和结构体中的static：

在类的所有实例中，这个变量只有一个实例。

**静态函数只能访问静态变量**

**每个非静态方法总是获取当前类的一个实例作为参数**

静态函数跟在类外编写函数相同。

```c++
struct Entity{
    static int x, y;
    
    void print()
    {
        std::cout << x << ' ' << y << std::endl;
    }
}

int Entity::x;
int Entity::y;

int main()
{
    Entity e;
    e.x = 2;
    e.y = 3;
    
    Entity e2;
    e2.x = 5;
    e2.y = 8;
    
    e.print();  //5，8
    e2.print();  //5，8
    
    Entity::x = 5;
    Entity::y = 8;  //对于静态变量的修改，应该这样子修改.
    
    std::cin.get();
}
```



## 类

### 类和结构体有什么区别？

默认情况下，类是private的；结构体是public的。

结构体的保留是为了c语言的向后兼容性。

### Header File vs Source Files

```c++
//.h file
#include <string>
class Student {
private:
    std::string name;
    std::string state;
    int age;
public:
    /// constructor for our student
    Student(std::string name, std::string state, int age);
    /// method to get name, state, and age, respectively
    std::string getName();
    Std::string getState();
    int getAge();
}
```

```c++
//.cpp file
#include “Student.h”
#include <string>
/// implement constructor
Student::Student(std::string name, std::string state, int age) {
    this->name = name;
    this->state = state;
    this->age = age;
}
```

* Header File  -->  **definition**
* Source File --> **implementations**

### Namespaces

* 每个类都有自己的命名空间
* `namespace_name::name`
* `std::string Student::getName() {...}`, 我们可以在这个函数里面使用私有成员变量

### 构造函数

* **默认构造函数**

  ```c++
  class A
  {
      public:
      	A() = default;
      	A(int B) {
              b = B;
          }
      private:
      	int b;
  }
  ```

  * 如果一个类中自定义带了参数的构造函数，那么编译器就不会再自动生成默认构造函数，也就是类将不能默认创建对象，只能携带参数进行创造对象。
  
  ```c++
  Student::Student() {
      name = “John”;
      state = “Appleseed”;
      age = 18;
  }
  ```
  
* **List initialization constructor (C++11)**

  `Student::Student(std::string name, std::string state, int age) name{name}, state{state}, age{age} {}`

### 析构函数

```c++
Student::~Student() {
    /// free/deallocate any data here
    delete [] my_array; /// for illustration
}
```

### Type aliasing

```c++
class Student {
Private:
    /// An example of type aliasing
    using String = std::string;
    String name;
    String state;
    int age;
}
```

### 虚函数

使用指针调用子类/父类的同名函数。。

```c++
class Entity
{
public:
	virtual std::string GetName() 
	{
		return "Entity";
	}	
};

class Player : public Entity
{
private:
	std::string m_name;
public:
	Player(const std::string & name) : m_name(name) {
	}
	
	std::string GetName() override {
		return m_name;
	}
};
```

```c++
Entity* e = new Entity;
Player* p = new Player("Alice");  //有参构造
std::cout << e->GetName() << std::endl;  //Entity
std::cout << p->GetName() << std::endl;  //Alice
Entity* e2 = p;
std::cout << e2->GetName() << std::endl;  //Alice
//如果不写成虚函数，最后一个输出就是Entity
```

### 纯虚函数（接口）

在基类定义一个没有实现的函数，强制子类去实现该函数。

只能在实现了所有纯虚函数之后，才能实例化类。

`virtual std::string GetName() = 0;`

### 隐式转换和explicit关键字

```c++
class Entity
{
private:
	std::string name;
	int age;
public:
	Entity(const std::string & name)
		:name(name), age(-1) {}
	
	Entity(int age):name("Unknow"), age(age) {}
	
	virtual std::string GetName() 
	{
		return "Entity";
	}	
};

int main()
{
    Entity* e = new Entity(10);
    Entity a(12);
    Entity b = 12;  //隐式转换
}
```

在构造函数前面加上`explicit`，就不能进行隐式转换了

### 特殊成员函数

**当它们被调用时才会生成。**

* Default constructor ：
* Destructor 
* Copy constructor ：create
* Copy assignment operator ：两个都已存在，将一个复制到另一个身上
* Move constructor 
* Move assignment operator

![image-20240321123405126](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403211234695.png)

* `default`  保留默认特殊成员函数
* `delete` 移除默认特殊成员函数（但是不能用在destructor上）

**默认移动构造函数**生成条件

* 没有复制构造函数被声明
* 没有移动构造函数被声明
* 没有析构函数被声明

## 模板

```c++
//container.h
template <typename T>
class Container {
public:
    Container(T val);
    T getValue();
private:
    T value;
};

#include "Container.cpp"
```

* ==要引入实现的cpp文件==

**模版函数的编写**

```c++
// Container.cpp
#include "container.h"

template <typename T>
Container<T>::Container(T val) {
    this->value = val;
}

template <typename T>
T Container<T>::getValue() {
    return value;
}
```

* c++希望我们在命名空间指定模板参数！！

  <img src="https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403191554914.png" alt="image-20240319155406149" style="zoom:50%;" />



* 当我们**调用函数**时，基于传递的参数，这个函数**才被创建**出来，并作为源代码被编译。

* 在定义类模板或者函数模板时，`typename` 和 `class` 关键字都可以用于指定模板参数中的类型。也就是说，以下两种用法是完全等价的。这在大多数文章中都有提到。

```c++
template<typename T> /* class or function declaration */;
template<class T>    /* class or function declaration */;
```

### template functions

```c++
template <typename T>
T myMin(T a, T b)
{
  return a < b ? a : b;
}
```

`template <typename T = int>` 默认类型

- `myMin<int>(3, 4)`调用模板函数
- `myMin(3, 4)`这样调用也行，交给编译器自己判断

```c++
template <typename T, typename U>
auto smarterMyMin(T a, U b) {
  return a < b ? a : b;
}

std::cout << smarterMyMin(3.2, 4) << std::endl;
```



### 其他模板

* 变量模板

  ```c++
  // 变量模板
  template<typename T>
  T zero = 0;
  
  auto i = zero<int>; // 相当于int i = 0;
  ```

* 别名模板

  ```c++
  // 别名模板
  template<typename T>
  using Container = std::vector<T>;
  
  Container<int> v{ 1,2,3 }; // 相当于std::vector<int> v{1, 2, 3};
  ```

  

### Concepts and Constraints

c++20才支持

Concepts是用来**约束模板类型**的语法糖

```c++
//定义一个concept
//只能是整数类型
template <typename T>
concept integral = std::is_integral_v<T>;

//使用一个concept
template <integral T>
T add(T a, T b) {
  return a + b;
}
```

### template metaprogramming(TMP)

模板元编程编程的主要目的，就是**利用编译期完成计算**，尽可能**减少运行时间**，从而达到减少开销的效果。

**模板元编程**

* 元编程类似于递归

* 模板的**特化**版本

  ```c++
  //假设想对所有类型返回较大值，但偏偏要对int类型返回较小值
  template<> // template<>表示这是一个特化版本
  int max<int>(int x, int y) // 加上一个尖括号并指定特化类型，如果可以推断出也可以省略
  {
      return x < y ? x : y; // 返回较小值
  }
  
  std::cout << ::max<int>(10, 20); // 现在将打印10
  std::cout << ::max(10, 20); // T推断为int，将打印10
  ```

```c++
template <unsigned n>
struct Factorial {
  enum { value = n * Factorial<n - 1>::value};
};

//特化版本的模板
template<>
struct Factorial<0> {
  enum { value = 1};
};
```

* 因为常量表达式最初只能用enum来声明

### constexpr

**将运算放在编译阶段**

```c++
constexpr double fib(int n) {
  if (n == 1) return 1;
  return fib(n - 1) * n;
}

int main() {
  const long long bigval = fib(20);
  std::cout << bigval << std::endl;
}
```



## Lambda 表达式

作用域中（incline 内联的）、匿名函数

``` c++
auto var = [capture-clause] (auto param) -> bool {}
```

* []：捕获列表，获取当前作用域中的变量
* ()：传递的参数
* -> type : 返回值类型
* []填入&可以包含当前作用域中/**类**的所有成员变量

![image-20240319205431602](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403192054766.png)

### functor

仿函数functor，就是使一个类的使用看上去象一个函数。其实现就是类中实现一个operator，这个类就有了类似函数的行为，就是一个仿函数类了。**它可以有成员变量保存状态。**

```c++
class functor {
public:
  int operator() (int arg) const {
    return num + arg;
  };

  functor(int num): num{num} {};
private:
  int num;
};

functor f(3);
std::cout << f(5);  //像函数一样使用这个对象
```

### std::function

`std::function<return_type(param_types)> func`

```c++
void print1(){
    std::cout << "hello, print1" << std::endl;
}

void print2(){
    std::cout << "hello, print2" << std::endl;
}

int main(int argc, char *argv[])
{
    std::function<void()> func(&print1);
    func();

    func = &print2;
    func();

    return 0;
}
```

`std::function`对c++中各种可调用实体（普通函数，函数指针，Lambda表达式，伪函数等）进行封装，形成一个新的可调用的对象

## 运算符重载

```c++
class Student {
    public:
    std::string getName() const;
    void setName(string name);
    int getAge() const;
    void setAge(int age);
    bool operator < (const Student& rhs) const;
    
    private;
    std::string name;
    std::string state;
    int age;
}
```

```c++
#include "student.h"
std::string Student::getName() const {
    ……
}

bool operator < (const Student& rhs) const {
    return age < rhs.age;
}
```

![image-20240320145821403](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403201458614.png)

* 友元函数可以访问类的公有、受保护以及**私有**成员！

## 移动语义

**优点**：数据的所有权可以被转移，从而避免了拷贝的过程，这可以大大提高程序的性能。

**引入**

```c++
vector<string> findAllWords(int i);
int main() {
 vector<string> words1 = findAllWords(12345); // (1)
 vector<string> words2; // (2)

 words2 = findAllWords(23456); // (3)
}
```

* （1）：`copy constructor`
* （2）：`default constructor`
* （3）：`copy assignment constructor`
* 在(3)这里，生成了两个vector，我们需要一个办法，直接将findAllwords的结果赋给words2
* 如果使用移动构造器呢？下述代码是正确的。

```c++
int main() {
 vector<string> words1 = findAllWords(12345);  //move constructor
 vector<string> words2;

 words2 = findAllWords(23456);  //move constructor
}
```

* 但是下面这份就不正确了！！

  ```c++
  vector<string> words1 = findAllWords(12345);
  vector<string> words2 = words1;  //把words1的空间偷走了！！
  ```

### 左值和右值

![image-20240321135113869](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403211351134.png)

### std::move

**将右值变成左值**

==除了类的定义实现，其他地方最好不要用这个函数！==



```c++
vector<T>& operator=(vector<T>&& other)
{
	if (&other == this) return *this;
	_size = std::move(other._size);
	_capacity = std::move(other._capacity);
 
	//we can steal the array
	delete[] _elems;
	_elems = std::move(other._elems);
	return *this;
}

vector<T>& v = {1, 2, 3};
vector<T>& v2;
v2 = std::move(v);
```

* 右值引用 `&&`

  普通的引用，是一个左值

  ```c++
  void add(int & x) {
      x = x + 1;
  }
  
  int x = 5;
  add(x);  //这是可以的，x是个左值
  add(5);  //这是不可以的，5是个右值
  ```

* 析构函数、复制构造函数、移动构造函数：**如果你自定义了某一个，其他两个都要一起自定义。**（因为你自定义了某一个，通常都是涉及了动态内存分配）

## std::optional

表示一个可能有值的对象

```c++
void main(){
    std::optional<int> num1 = {}; //num1 does not have a value
    num1 = 1; //now it does!
    num1 = std::nullopt; //now it doesn't anymore
}
```

* `.value()`：取值，没有值则返回`bad_optional_access`erro

* `.has_value()`：判断是否有值

* 构造

  ```c++
  std::optional<int> val7 = std::make_optional<int>(128);
  std::optional<int> val6 = {128};
  ```

## RAII

**RAII: Resource Acquisition is Initialization**

* 类所有的资源都要在构造函数中获取
* 类所有的资源都要在析构函数中释放

对比下面两份代码

```c++
RawResourceHandle* handle=createNewResource();
handle->performInvalidOperation();  // Oops, throws exception
...
deleteResource(handle); // oh dear, never gets called so the resource leaks
```

* 还没运行`deleteResource`函数，就抛出异常，发生内存溢出。

使用RAII

```c++
class ManagedResourceHandle {
public:
   ManagedResourceHandle(RawResourceHandle* rawHandle_) : rawHandle(rawHandle_) {};
   ~ManagedResourceHandle() {delete rawHandle; }
   ... // omitted operator*, etc
private:
   RawResourceHandle* rawHandle;
};

ManagedResourceHandle handle(createNewResource());
handle->performInvalidOperation();
```

* 当抛出异常，就会自动调用析构函数，释放资源，不会发生内存溢出。

**RAII for locks → lock_guard**

**RAII for memory → smart pointers**

## 智能指针

`#include <memory>`

不再使用new和delete

* `unique_ptr`是作用域指针，超出作用域，自动delete，不能复制（因为当两个`unique_ptr`指向同一块内存，其中一个delete释放内存，另一个就会指向被释放的内存）栈分配

  使用`make_unique`创建

  `std::unique_ptr<Entity> entity = std::make_unique<Entity>("Alice", 12);`

  **参数传递**为生成类型的**构造函数参数**

  ```c++
  class Entity
  {
  private:
  	std::string name;
  	int age;
  public:
  	Entity(const std::string & name, int age):name(name), age(age){
  	}	
  };
  
  int main()
  {
      
      {
      	std::unique_ptr<Entity> entity = std::make_unique<Entity>("Alice", 12);
  	}
  }
  ```

* `shared_ptr`引用计数，引用数为0时会delete

  `std::shared_ptr<Entity> entity = std::make_shared<Entity>();`
  
  ![image-20240321154716279](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403211547833.png)

## dynamic_cast

动态类型转换，**沿继承层次**进行的强制类型转换

 通常**用于验证**

我们可以尝试在实体Entity对象上进行`dynamic_cast`，将其转换为玩家对象`Player`对象，如果返回null就不是`Player`

```c++
class Entity
{
public:
	virtual void GetName(){
	}	
};

class Player : public Entity
{
};

class Enemy : public Entity
{
};

int main()
{
	Entity * actuallyPlayer = new Player();
	Entity * actuallyEnemy = new Enemy();
	
	Player* p0 = dynamic_cast<Player*>(actuallyPlayer);
	Player* p1 = dynamic_cast<Player*>(actuallyEnemy);
	std::cout << p1;  //0 就是nullptr
}
```



## 后置返回类型

```c++
// 传统的函数声明方式

int func(int a, int b);

// 使用后置返回类型

auto func(int a, int b) -> int;
```

此外，后置返回类型还可以使函数模板更加容易定义。在函数模板中，返回类型通常需要使用模板参数，这会使函数声明变得复杂。而使用后置返回类型可以将返回类型与模板参数分开，使其更加清晰。例如：

```c++
// 传统的函数模板声明方式

template<typename T>

int func(T a, T b);

// 使用后置返回类型

template<typename T>

auto func(T a, T b) -> int;
```

此外，后置返回类型还可以使函数声明更加灵活。在某些情况下，返回类型可能需要使用其它变量或表达式计算得出。使用后置返回类型可以将这些计算过程放在箭头（->）后，使得函数声明更加灵活。例如：

```c++
auto func(int a, int b) -> decltype(a + b);
```

## std::lock_guard

用法是在函数开始的地方声明一个lock_guard 对象，构造函数中启用加锁，函数结束的时候，这个lock_guard 对象作用域也就结束了，自动析构，析构时会自动释放锁！

```c++
std::lock_guard<std::mutex> lock(latch_);
```

在函数内调用其他函数怎么办？

递归锁运行一个线程对同一个互斥量多次加锁

## std::condition_variable

先看一下下面这份代码理解一下

```c++
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx; // 全局互斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false; // 全局标志位.

void do_print_id(int id)
{
    std::unique_lock <std::mutex> lck(mtx);
    while (!ready) // 如果标志位不为 true, 则等待...
        cv.wait(lck); // 当前线程被阻塞, 当全局标志位变为 true 之后,
    // 线程被唤醒, 继续往下执行打印线程编号id.
    std::cout << "thread " << id << '\n';
}

void go()
{
    std::unique_lock <std::mutex> lck(mtx);
    ready = true; // 设置全局标志位为 true.
    cv.notify_all(); // 唤醒所有线程.
}

int main()
{
    std::thread threads[10];
    // spawn 10 threads:
    for (int i = 0; i < 10; ++i)
        threads[i] = std::thread(do_print_id, i);

    std::cout << "10 threads ready to race...\n";
    go(); // go!

  for (auto & th:threads)
        th.join();

    return 0;
}

```

* **当前线程调用 wait() 后将被阻塞**
* 在线程被阻塞时，该函数会自动调用 **lck.unlock()** 释放锁，使得其他被阻塞在锁竞争上的线程得以继续执行。另外，一旦当前线程获得通知(notified，通常是另外某个线程调用 notify_* 唤醒了当前线程)，wait() 函数也是自动调用 lck.lock()，使得 lck 的状态和 wait 函数被调用时相同。

## volatile

volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改。比如：**操作系统、硬件或者其它线程**等。遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。

```c++
volatile int i = 10;
while (i == 10)
{
    // code
}
```

volatile 指出 i 是随时可能发生变化的，每次使用它的时候必须从 i的地址中读取。

不加的话，i每次就会从寄存器中读取。
