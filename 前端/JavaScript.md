# JavaScript

## JavaScript基础

### JavaScript历史

* 1994年,网景公司(Netscape)发布了Navigator浏览器0.9版
* 网景公司决定与Sum公司结成联盟,希望将Java嵌入网页中
* Brendan Eich设计出来了JavaScript,跟Java没啥关系,蹭蹭热度
* ECMAScript是一种规范,而JavaScript是这种规范的一种实现

### JavaScript运行引擎

* JavaScriptCore: WebKit中的JavaScript引擎,Apple公司开发
* V8: Google开发的JavaScript引擎

### JavaScript的编写方式

* 编写在script元素之内:`<script></script>`
* 外部的script文件

### noscript元素的使用

* 对于不支持JavaScript的浏览器,给用户一个提示

### JavaScript交互方式

* `console.log` 接受多个参数
* `alert` 接受一个参数
* `prompt`在浏览器接受用户输入

### JavaScript注释

* //
* /**/

## 变量和数据类型

### 常见数据类型

在JavaScript中有8种基本的数据类型(7种原始类型和1种复杂类型)

* Number
  * NaN: 代表一个计算错误
  * Infinity: 无穷大
* String
* Boolean
* Undefined
* Null : 一个**对象**为空, 对象初始化
* Object
* BigInt
* Symbol

### 数据类型的转换

* String() 或 +操作
* Number() 或 */操作
* Boolean()

## 运算符

* `===`和`==`的区别: `==`会进行类型转换, 而`===`不会做任何类型转换

  ```
  0 == false  //返回true
  0 === false  //返回false
  ```

  

## 函数

### foo, bar, baz

* 本身没有特别的用途和意义
* 伪变量

### 函数的声明和调用

```javascript
function 函数名() {
    
}
```

```javascript
函数名()
```

```javascript
function printInfo(name, age) {
    console.log('my name is ${name}')
    console.log('age is ${age}')
}
```

* 返回值: 如果没有写return, 返回**undefined**

### arguments变量

```javascript
function foo(name, age) {
    console.log(arguments)
}
```

* arguments 是个对象!
* `for (var i = 0; i < arguments.length; i ++)` 遍历, 不是数组!

### 局部和全局变量

* ES5之前是**没有块级作用域**的概念的, var定义的变量是没有块级作用域的

### 函数表达式的写法

```javascript
var bar = function() {
    
} //允许省略函数名
```

```javascript
foo()
function foo(){
    
}

//这个是可以的!
```

* JavaScript准备运行脚本的时候,首先会在脚本中寻找全局函数声明,并创建这些函数

```javascript
foo()
var foo = function() {
    
}

//这个是不可以的!
```

* 函数表达式是在代码执行到达时创建

### 函数的头等公民

JavaScript是一种函数式编程语言.

1. 函数可以被赋值给变量
2. 函数可以作为另外一个函数的参数
3. 函数可以作为另一个函数的返回值
4. 可以将函数存储在另一个数据结构里面

### 高阶函数

* 接受一个或多个函数作为输入
* 输出一个函数

### 立即执行函数的使用(用到再查,很少用了)

一个函数在定义完之后被立即执行

```javascript
(function(){
    console.log("")
})()
```

#### 应用

1. 防止全局变量命名冲突
2. 在btns的点击中,创建一个作用域保存值 (ES6 使用let 就可以解决了)

## 对象

### 对象类型

* 存储键值对

* 对象中的函数称为**方法**

### 对象的创建方式

1. 对象字面量

   ```javascript
   var obj = {
       name: "xxx"
   }
   ```

2. new Object

3. new 自定义类()

### 对象的操作过程

1. 访问: `console.log(obj.name)`
2. 修改: `obj.name = 'xxx'`
3. 添加: 有就修改,没有就添加
4. 删除: `delete info.age`

### 对象的遍历方式

* `Object.keys(obj)` 返回对象key组成的数组
* `for..in..`: `for(var key in info)` 

## 内存分配

* 原始类型占据的空间是在**栈内存**中分配的 - 值类型
* 对象类型占据的空间是在**堆内存**中分配的 - 引用类型

![image-20230412205300941](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20230412205300941.png)

学会画图!

day22_13 非常值得一听!

## this的使用

* 函数中有个this变量, arguments保存的是传入的所有参数, this变量大多数情况下会指向一个**对象**

1. this的绑定和调用方式以及调用的位置有关, this是在**运行时**被绑定的

2. 三条绑定规则

   * 默认绑定: 如果普通函数被默认调用, this指向window

   * 隐式绑定: 如果一个函数被某一个对象并且调用它, this指向该对象

     ```javascript
     obj.running()  //obj
     
     var fn = obj.running
     fn()  //window
     ```

   * new绑定: 指向新创建的对象

   * 显式绑定: 

     ```javascript
     function foo() {
         console.log("foo函数被调用", this)
     }
     
     foo()
     foo.apply(obj)  //函数的this就会指向obj
     foo.call(obj)
     
     function foo(name, age)
     {
         console.log("foo函数被调用", this)
         console.log(name, age)
     }
     //区别: 传参的方式不同
     foo.apply(obj, ["Alice", 18])
     foo.call(obj, "Alice", 18)
     ```

     绑定函数(bind)

     ```JavaScript
     var obj = {name: "why"}
     var bar = foo.bind(obj)
     bar()
     ```

3. 规则优先级: new > bind/apply/call > 隐式绑定 > 显式绑定

4. 箭头函数没有this和arguments, 箭头函数不能做构造函数

   ```javascript
   var names = ["abc", "ccc", "123"]
   names.forEach((item, index, arr) => {
       console.log(item, index, arr)
   })
   
   //如果箭头函数只有一个参数, 小括号可以省略
   names.forEach(item => {
       console.log(item)
   })
   
   //如果函数体中只有一行执行代码, 大括号可以省略
   names.forEach(item => console.log(item))
   
   names.forEach(item => "123")  //没有return, 箭头指向的值就是返回值
   
   var obj = {
       name: "obj", 
       foo: () => {
           var bar = () => {
               console.log("bar", this)
           }
           return bar
       }
   }
   
   var fn = obj.foo()
   fn.apply("bbb")  //通过apply调用箭头函数也没有this
   //此时this返回的是window, this函数就是一层一层向外找, 找到有this的函数, 往上层作用域找, 注意对象是数据类型不是作用域
   ```



练习题

```javascript
1. 
var name = "window";
var person = {
  name: "person",
  sayName: function () {
    console.log(this.name);
  }
};

function sayName() {
  var sss = person.sayName;
  sss(); 
  person.sayName(); 
  (person.sayName)(); 
  (b = person.sayName)(); 
}

//答案
function sayName() {
  var sss = person.sayName;
  sss(); // window
  person.sayName(); // person
  (person.sayName)(); // person
  (b = person.sayName)(); // window 间接函数引用
}
sayName();

2.
var name = 'window'
var person1 = {
  name: 'person1',
  foo1: function () {
    console.log(this.name)
  },
  foo2: () => console.log(this.name),
  foo3: function () {
    return function () {
      console.log(this.name)
    }
  },
  foo4: function () {
    return () => {
      console.log(this.name)
    }
  }
}

var person2 = { name: 'person2' }

person1.foo1(); 
person1.foo1.call(person2); 

person1.foo2();
person1.foo2.call(person2);

person1.foo3()();
person1.foo3.call(person2)();
person1.foo3().call(person2); 

person1.foo4()(); 
person1.foo4.call(person2)(); 
person1.foo4().call(person2); 

// person1.foo1(); // person1
// person1.foo1.call(person2); // person2

往上层作用域找, 对象不是作用域!
// person1.foo2(); // window
// person1.foo2.call(person2); // window

// person1.foo3()(); //隐式绑定 window
// person1.foo3.call(person2)(); //隐式绑定 window
// person1.foo3().call(person2); //显式绑定 person2

// person1.foo4()(); // person1, 往上层作用域找, 就找到了person1.foo4()这个函数,它绑定的this是person1
// person1.foo4.call(person2)(); // person2, 同理,往上层作用域找,上层作用域通过显示绑定person2
// person1.foo4().call(person2); // person1, this的显式绑定,绑了跟没绑一样,直接往上层作用域找,找到person1

3.
var name = 'window'
function Person (name) {
  this.name = name
  this.foo1 = function () {
    console.log(this.name)
  },
  this.foo2 = () => console.log(this.name),
  this.foo3 = function () {
    return function () {
      console.log(this.name)
    }
  },
  this.foo4 = function () {
    return () => {
      console.log(this.name)
    }
  }
}
var person1 = new Person('person1')
var person2 = new Person('person2')

person1.foo1() 
person1.foo1.call(person2) 

person1.foo2() 
person1.foo2.call(person2) 

person1.foo3()()
person1.foo3.call(person2)()
person1.foo3().call(person2) 

person1.foo4()() 
person1.foo4.call(person2)() 
person1.foo4().call(person2) 

// person1.foo1() // person1
// person1.foo1.call(person2) // person2

// person1.foo2() // person1
// person1.foo2.call(person2) // person1

// person1.foo3()() // window
// person1.foo3.call(person2)() // window
// person1.foo3().call(person2) // person2

// person1.foo4()() // person1
// person1.foo4.call(person2)() // person2
// person1.foo4().call(person2) // person1

4.
var name = 'window'
function Person (name) {
  this.name = name
  this.obj = {
    name: 'obj',
    foo1: function () {
      return function () {
        console.log(this.name)
      }
    },
    foo2: function () {
      return () => {
        console.log(this.name)
      }
    }
  }
}
var person1 = new Person('person1')
var person2 = new Person('person2')

person1.obj.foo1()() 
person1.obj.foo1.call(person2)() 
person1.obj.foo1().call(person2) 
person1.obj.foo2()() 
person1.obj.foo2.call(person2)()
person1.obj.foo2().call(person2) 

// person1.obj.foo1()() // window
// person1.obj.foo1.call(person2)() // window
// person1.obj.foo1().call(person2) // person2

// person1.obj.foo2()() // obj
// person1.obj.foo2.call(person2)() // person2
// person1.obj.foo2().call(person2) // obj
```



## 类和对象(ES5)

**JavaScript中构造函数扮演了其他语言中类的角色**

(在ES6之后, JavaScript可以像别的语言一样, 通过class来声明一个类)

```javascript
function Person(name, age, height)
{
    this.name = name
    this.age = age
    this.height = height
    this.eating = function() {
        console.log(this.name + "在吃饭~")
    }
}

var p1 = new Person("Mark", 19, 172)
```

如果一个函数被使用new操作符调用了, 那么它会执行如下操作:

1. 在内存创建一个空对象
2. 构造函数内部的this, 会指向创建出来的新对象
3. 执行函数代码
4. 如果构造函数没有返回非空对象, 则返回创建出来的空对象



补充: 函数也是一个对象, Function extends Object, 存在堆内存中

## DOM

DOM: 文档对象模型

BOM: 浏览器对象模型(navigator, screen, history...)

### DOM元素之间的关系

* **节点**之间的关系
  * 父节点: parentNode
  * 前兄弟节点: previousSibling
  * 后兄弟节点: nextSibling
  * 子节点: childNodes
  * 第一个子节点: firstChild
  * 最后一个子节点: lastChild
* **元素**之间的关系(更常用)
  * 父元素: parentElement
  * 前兄弟元素: previousElementSibling
  * 后兄弟元素: nextElementSibling
  * 子节点: children
  * 第一个子节点: firstElementChild
* 表格元素
  * table.rows
  * tr.cells

### 获取DOM元素

* 最常用的两个!!!

  * querySelector

  * querySelectorAll

    举例: `document.querySelector("#xxx")`

* 其他, 不太常用

  * getElementById (只能在document上使用)
  * getElementByClassName

* 补充:

  * `var keywordEl = document.querySelectorAll(".keyword")` keywordEl 是一个可迭代的**对象**! 不是数组, 是**NodeList**, 类数组对象, 可以使用`keywordEl[0]`这种操作, 也可以使用`for of`操作.

### DOM节点的type, tag, content

* nodeType: 节点类型
* nodename: 节点名称
* tagname: 元素标签名称, 对于元素来说, tagname和nodename相同
* innerHTML: 将**元素**中的HTML获取为字符串形式
* textContent: 仅仅获取**元素**中的文本内容
* data: 获取**非元素节点**的内容

### DOM节点的attributes, properies

* **元素**的属性叫 attributes, **对象**的属性叫 properies
* attributes: 
  * 标准: id, class, href, type ...
  * 非标准: 自定义属性
  * 值总是是**字符串类型**的
  * `el.hasAttribute(name)`, `el.getAttribute()`, `el.setAttribute(name, value)`(现用现查吧)
* property:
  * 对于标准的attributes, 会在DOM对象上创建与其对应的property属性
  * 默认情况下是**有类型**的
* 获取非标准属性: data-*
  * 在dataset属性中获取, 小程序中常常使用!

```html
<div class="box" data-name="why" age="19" id="ii"></div>

<script>
	var boxel = document.querySelector(".box");
    console.log(boxel.hasAttribute("age"));
    
    boxel.setAttribute("age", "11");  //使用Attributes
    
    boxel.id = "jjj";  //使用property
    
    console.log(boxel.dataset.name);  //使用data-*
</script>
```



### DOM节点的创建/插入/克隆/删除

```javascript
var boxel = document.querySelector(".box");
var h2el = document.createElement("h2");  //元素创建
h2el.innerHTML = "Title";
boxel.append(h2el);  //元素插入, 具体查看文档

setTimeout(() => {
    h2el.remove();  //元素删除
}, 2000);

var cloneBoxel = boxel.cloneNode(true);  //元素的克隆
document.body.prepend(cloneBoxel);
```



### DOM节点的样式, 类

* 元素的class attribute, 对应的property叫做 **className**
* classList: 
  * `el.classList.add(class)`
  * `el.classList.remove(class)`
  * 现用现查!
* `getComputedStyle(boxel).width` : 元素style的读取
* 只有**内联样式**, 才能使用 `el.style.xxx`

### DOM元素/window的大小, 滚动, 坐标

全是接口, 自查文档.

## 事件处理

* 事件监听方式

  * DOM属性, 通过元素的on来监听事件
  * 通过EventTarget中的addEventListener来监听

* 事件冒泡和事件捕获

  * 事件冒泡: 事件从最内层向最外层传递
  * 事件捕获: 事件从最外层向最内层传递

  ```javascript
  divEl.addEventListener("click", function() {
  	console.log("div被点击");
  }, true);  //传入第二个参数, 就会变成事件捕获
  ```

* 事件对象

  * event对象会在传入事件处理函数回调是, 被系统传入

  ```javascript
  divEl.addEventListener("click", function(event) {
  	console.log("事件对象", event)
  })
  ```

  * 常见属性:
    * type: 事件类型
    * target: 当前事件**发生**的对象
    * currentTarget: 当前**处理事件**的对象

* EventTarget类:

  * 所有的节点、元素都继承自EventTarget
  * EventTarget是一个DOM接口， 主要用于添加、删除、派发Event事件

  ```javascript
  function fn() {
      console.log("函数")
  }
  
  divEl.addEventListener("click", fn);
  divEl.removeEventListener("click", fn);  //要指明删除哪个事件
  ```

* 事件委托: 当子元素被点击时, 父元素可以通过冒泡监听到子元素的点击, 并且可以通过event.target获取到当前监听的元素.

## debug调试技巧

* chrome - sources
* debugger 标识符在代码里面打断点

## 语法糖

* 10_0000_0000 表示 1000000000 提高数字的可阅读性



