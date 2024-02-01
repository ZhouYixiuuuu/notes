# 现代c++

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



## 引用

指针的语法糖

没有实际的物理内存，只是取了一个别名

函数通过**传递引用**才能实际修改参数值。

```c++
int a = 5;
int& ref = a;
a = 2;
std::cout << ref << std::endl;  //2

int f(int& a){}  //传递引用
```

## 类

### 类和结构体有什么区别？

默认情况下，类是private的；结构体是public的。

结构体的保留是为了c语言的向后兼容性。

### 虚函数

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

## const

## mutable

## 智能指针

`#include <memory>`

不再使用new和delete

* `unique_ptr`是作用域指针，超出作用域，自动delete，不能复制（因为当两个`unique_ptr`指向同一块内存，其中一个delete释放内存，另一个就会指向被释放的内存）栈分配

  使用`make_unique`创建

  `std::unique_ptr<Entity> entity = std::make_unique<Entity>("Alice", 12);`

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



## 左值和右值



## 移动语义



## std::move与移动赋值操作符



## 模板

```c++
template<typename T>
void Print(T value)
{
    std::cout << value << std::endl;
}

int main()
{
    Print(5);
    Print("Hello");
    Print(5.5f);
}
```

当我们调用函数时，基于传递的参数，这个函数才被创建出来，并作为源代码被编译。