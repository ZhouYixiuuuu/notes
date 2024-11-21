# CMake

![image-20240315154414383](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202403151544671.png)

## CMake使用

### 注释

CMake 使用 # 进行行注释，可以放在任何位置

CMake 使用 #[[ ]] 形式进行块注释。

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
add_executable(app add.cpp div.cpp main.cpp mult.cpp sub.cpp)
```

* `cmake_minimum_required`：指定使用的 cmake 的最低版本

  可选，非必须，如果不加可能会有警告

* `project`：定义工程名称，并可指定工程的版本、工程描述、web主页地址、支持的语言（默认情况支持所有语言），如果不需要这些都是可以忽略的，只需要指定出工程名字即可。
* `add_executable(可执行程序名 源文件名称)`

1. 执行`CMake`命令，生成makefile文件

   `cmake CMakeLists.txt文件所在路径`

   `cmake .`

   如果在`CMakeLists.txt`文件所在目录执行了`cmake`命令之后就会生成一些目录和文件（包括 `makefile` 文件），将这些文件放进`build`文件夹下面。

   ```shell
   $ mkdir build
   $ cd build
   $ cmake ..
   ```

   在build目录中执行`make` 命令

2. 执行`make`命令，生成可执行程序

#### 定义变量

```cmake
# 方式1: 各个源文件之间使用空格间隔
# set(SRC_LIST add.c  div.c   main.c  mult.c  sub.c)

# 方式2: 各个源文件之间使用分号 ; 间隔
set(SRC_LIST add.c;div.c;main.c;mult.c;sub.c)
add_executable(app  ${SRC_LIST})
```

#### 指定c++版本

```cmake
#增加-std=c++11
set(CMAKE_CXX_STANDARD 11)
#增加-std=c++14
set(CMAKE_CXX_STANDARD 14)
```

#### 搜索文件

如果一个项目文件太多，在编写CMakeLists.txt文件的时候不可能将所有文件一一列出。

```cmake
file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
```

* `GLOB`: 将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。
* `GLOB_RECURSE`：**递归搜索**指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中。

* `CMAKE_CURRENT_SOURCE_DIR` 宏表示当前访问的 CMakeLists.txt 文件所在的路径
* `PROJECT_SOURCE_DIR`宏对应的值就是我们在使用cmake命令时，后面紧跟的目录，一般是工程的根目录

```cmake
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
```

#### 包含头文件

```cmake
include_directories(headpath)
```

#### 静态库

```cmake
add_library(库名称 STATIC 源文件1 [源文件2] ...) 
```

#### 动态库

```cmake
add_library(库名称 SHARED 源文件1 [源文件2] ...) 
```

