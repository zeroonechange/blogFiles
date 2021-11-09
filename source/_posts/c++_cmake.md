---
title: cmake
date: 2021-11-10 01:51:37
tags: c++
---


#### 概述

```
cmake 和 make 
1.gcc是GNU Compiler Collection（就是GNU编译器套件），也可以简单认为是编译器，它可以编译很多种编程语言（括C、C++、Objective-C、Fortran、Java等等）。
2.当你的程序只有一个源文件时，直接就可以用gcc命令编译它。
3.但是当你的程序包含很多个源文件时，用gcc命令逐个去编译时，你就很容易混乱而且工作量大
4.所以出现了make工具
make工具可以看成是一个智能的批处理工具，它本身并没有编译和链接的功能，而是用类似于批处理的方式—通过调用makefile文件中用户指定的命令来进行编译和链接的。
5.makefile是什么？简单的说就像一首歌的乐谱，make工具就像指挥家，指挥家根据乐谱指挥整个乐团怎么样演奏，make工具就根据makefile中的命令进行编译和链接的。
6.makefile命令中就包含了调用gcc（也可以是别的编译器）去编译某个源文件的命令。
7.makefile在一些简单的工程完全可以人工手下，但是当工程非常大的时候，手写makefile也是非常麻烦的，如果换了个平台makefile又要重新修改。
8.这时候就出现了Cmake这个工具，cmake就可以更加简单的生成makefile文件给上面那个make用。当然cmake还有其他功能，就是可以跨平台生成对应平台能用的makefile，你不用再自己去修改了。
9.可是cmake根据什么生成makefile呢？它又要根据一个叫CMakeLists.txt文件（学名：组态档）去生成makefile。
10.到最后CMakeLists.txt文件谁写啊？亲，是你自己手写的。
11.当然如果你用IDE，类似VS这些一般它都能帮你弄好了，你只需要按一下那个三角形
12.cmake是make maker，生成各种可以直接控制编译过程的控制器的配置文件，比如makefile、各种IDE的配置文件。
13.make是一个简单的通过文件时间戳控制自动过程、处理依赖关系的软件，这个自动过程可以是编译一个项目。
```

#### Linux上安装实操

```bash
SSH工具   https://mobaxterm.mobatek.net/
查看系统型号 cat /proc/version
安装cmake   
    从官网下载 tar.gz文件  解压 tar zxf cmake-3.7.2.tar.gz    https://cmake.org/download/
    cd 解压目录    执行  ./bootstrap  
      检查配置  ./configura
	  安装  yum install gcc gcc-c++  (Cannot find a C++ compiler that supports both C++11 and the specified C++ flags)
	  安装  yum install openssl-devel 
	编译  make -j 4  别用8  内存爆了 
	make install
	cmake --version

编译
	在CMakeLists.txt目录下    mkdir build
						   cmake ..
						   make 
						   ./Demo 4 5 (单个文件里面是个幂函数 接受俩个参数)
https://www.hahack.com/codes/cmake/
```


#### 编译

```cmake
----------------------------------
单个文件编译
# cmake 最低版本要求
cmake_minimum_required (VERSION 2.8)   
# 项目信息
project (Demo1)
# 指定生成目标
add_executable(Demo main.cc)
----------------------------------
同目录多个文件编译
# 把当前目录下的所有源文件存入变量名
aux_source_directory(. DIR_SRC)
add_executable(Demo ${DIR_SRC})
----------------------------------
子目录文件编译
# 添加math子目录
add_subdirectory(math)
add_executable(Demo ${DIR_SRC})
# 添加链接库
target_link_libraries(Demo MathFunctions)
子目录的CMakeLists.txt:
#将当前目录源文件 保存到 变量
aux_source_directory(. DIR_LIB_SRC)
# 生成链接库   将 src 目录中的源文件编译为静态链接库
add_library (MathFunctions ${DIR_LIB_SRC})
----------------------------------
自定义编译-用自己的还是用系统的  加个控制变量 
 
# 是否使用自己的 MathFunctions 库
option (USE_MYMATH
	   "Use provided math implementation" ON)
# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
  "${PROJECT_SOURCE_DIR}/config.h.in"  # 由CMake将config.h.in 自动生成 config.h 文件
  "${PROJECT_BINARY_DIR}/config.h"
  )
config.h.in  只有一句  #cmakedefine USE_MYMATH

# 是否加入 MathFunctions 库
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/math")
  add_subdirectory (math)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

main.cc
#ifdef USE_MYMATH
  #include <MathFunctions.h>
#else
  #include <math.h>
#endif

#ifdef USE_MYMATH
    printf("Now we use our own Math library. \n");
    double result = power(base, exponent);
#else
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#endif
在linux下编译
mkdir build    -  cd build  
cmake -D  USE_MYMATH=OFF .. 
make
./Demo 3 4   -->  2 ^ 3 is 8
```


####  指定安装规则
```cmake
-------------------------------------
指定安装规则  添加测试   产生MakeFile后使用 make install 和 make test 来执行 
在子目录 math下修改CMakeLists.txt  
# 指定 MathFunctions 库的安装路径
install (TARGETS MathFunctions DESTINATION lib)
install (FILES MathFunctions.h DESTINATION include)

在根目录修改CMakeLists.txt
# 指定安装路径
install (TARGETS Demo DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h"
         DESTINATION include)
```
```bash
生成的 Demo 文件和 MathFunctions 函数库 libMathFunctions.o 文件将会被复制到 /usr/local/bin 中
而 MathFunctions.h 和生成的 config.h 文件则会被复制到 /usr/local/include 中
mkdir build  ->  cd build -> cmake ..  -> make install 

[root@iZwz97bu0gr8vx4fbs6n1uZ build]# make install
[ 25%] Building CXX object math/CMakeFiles/MathFunctions.dir/MathFunctions.cc.o
[ 50%] Linking CXX static library libMathFunctions.a
[ 50%] Built target MathFunctions
[ 75%] Building CXX object CMakeFiles/Demo.dir/main.cc.o
[100%] Linking CXX executable Demo
[100%] Built target Demo
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/Demo
-- Installing: /usr/local/include/config.h
-- Installing: /usr/local/lib/libMathFunctions.a
-- Installing: /usr/local/include/MathFunctions.h
```

#### 添加测试
```cmake
添加测试  CTest 
# 启用测试
enable_testing()
# 测试程序是否成功运行
add_test (test_run Demo 5 2)
# 测试帮助信息是否可以正常提示
add_test (test_usage Demo)
set_tests_properties (test_usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")# 启用测试
enable_testing()
# 测试程序是否成功运行
add_test (test_run Demo 5 2)
# 测试帮助信息是否可以正常提示
add_test (test_usage Demo)
set_tests_properties (test_usage
        PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")
# 测试 5 的平方
add_test (test_5_2 Demo 5 2)
set_tests_properties (test_5_2
        PROPERTIES PASS_REGULAR_EXPRESSION "is 25")
# 测试 10 的 5 次方
add_test (test_10_5 Demo 10 5)
set_tests_properties (test_10_5
        PROPERTIES PASS_REGULAR_EXPRESSION "is 100000")
# 测试 2 的 10 次方
add_test (test_2_10 Demo 2 10)
set_tests_properties (test_2_10
        PROPERTIES PASS_REGULAR_EXPRESSION "is 1024")
```
```bash
[root@iZwz97bu0gr8vx4fbs6n1uZ build]# make test
Running tests...
Test project /c++/workspace/demo5/build
    Start 1: test_run
1/5 Test #1: test_run .........................   Passed    0.00 sec
    Start 2: test_usage
2/5 Test #2: test_usage .......................   Passed    0.00 sec
    Start 3: test_5_2
3/5 Test #3: test_5_2 .........................   Passed    0.00 sec
    Start 4: test_10_5
4/5 Test #4: test_10_5 ........................   Passed    0.00 sec
    Start 5: test_2_10
5/5 Test #5: test_2_10 ........................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 5

Total Test time (real) =   0.02 sec
```

简化
```cmake
# 定义一个宏，用来简化测试工作
macro (do_test arg1 arg2 result)
  add_test (test_${arg1}_${arg2} Demo ${arg1} ${arg2})
  set_tests_properties (test_${arg1}_${arg2}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)

# 利用 do_test 宏，测试一系列数据
do_test (5 2 "is 25")
do_test (10 5 "is 100000")
do_test (2 10 "is 1024")

支持 gdb调试
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

```cmake
环境检查
是否自带 pow 函数。如果带有 pow 函数，就使用它；否则使用我们定义的 power 函数
在根目录CMakeLists 添加 CheckFunctionExists 宏
# 检查系统是否支持 pow 函数
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
check_function_exists (pow HAVE_POW)

预定义相关宏变量    修改 config.h.in 文件
#cmakedefine HAVE_POW

在代码中使用宏和函数
#ifdef HAVE_POW
  #include <math.h>
#else
  #include <MathFunctions.h>
#endif

#ifdef HAVE_POW
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#else
    printf("Now we use our own Math library. \n");
    double result = power(base, exponent);
#endif

添加版本号  在根目录CMakeLists 设置主版本号和副版本号
# 项目信息
project (Demo7)
set (Demo_VERSION_MAJOR 1)
set (Demo_VERSION_MINOR 0)

为了在代码中获取版本信息，我们可以修改 config.h.in 文件，添加两个预定义变量
#define Demo_VERSION_MAJOR @Demo_VERSION_MAJOR@
#define Demo_VERSION_MINOR @Demo_VERSION_MINOR@
在Main.cc打印版本信息 
    if (argc < 3){
        // print version info
        printf("%s Version %d.%d\n",
            argv[0],
            Demo_VERSION_MAJOR,
            Demo_VERSION_MINOR);
        printf("Usage: %s base exponent \n", argv[0]);
        return 1;
    }
```
#### 生成安装包
```cmake
生成安装包   CPack  配置生成各种平台上的安装包  
在根目录CMakeLists 
# 构建一个 CPack 安装包
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE
  "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${Demo_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Demo_VERSION_MINOR}")
include (CPack)

1.导入 InstallRequiredSystemLibraries 模块，以便之后导入 CPack 模块；
2.设置一些 CPack 相关变量，包括版权信息和版本信息，其中版本信息用了上一节定义的版本号；
3.导入 CPack 模块。

mkdir build  ->  cd build -> cmake .. 

```

```bash
# 生成二进制安装包
[root@iZwz97bu0gr8vx4fbs6n1uZ build]# cpack -c CPackConfig.cmake
CPack: Create package using STGZ
CPack: Install projects
CPack: - Run preinstall target for: Demo8
CPack: - Install project: Demo8 []
CPack: Create package
CPack: - package: /c++/workspace/demo8/build/Demo8-1.0.1-Linux.sh generated.
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: Demo8
CPack: - Install project: Demo8 []
CPack: Create package
CPack: - package: /c++/workspace/demo8/build/Demo8-1.0.1-Linux.tar.gz generated.
CPack: Create package using TZ
CPack: Install projects
CPack: - Run preinstall target for: Demo8
CPack: - Install project: Demo8 []
CPack: Create package
CPack: - package: /c++/workspace/demo8/build/Demo8-1.0.1-Linux.tar.Z generated.

# 生成源码安装包
[root@iZwz97bu0gr8vx4fbs6n1uZ build]# cpack -C CPackSourceConfig.cmake
CPack: Create package using STGZ
CPack: Install projects
CPack: - Run preinstall target for: Demo8
CPack: - Install project: Demo8 [CPackSourceConfig.cmake]
CPack: Create package
CPack: - package: /c++/workspace/demo8/build/Demo8-1.0.1-Linux.sh generated.
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: Demo8
CPack: - Install project: Demo8 [CPackSourceConfig.cmake]
CPack: Create package
CPack: - package: /c++/workspace/demo8/build/Demo8-1.0.1-Linux.tar.gz generated.
CPack: Create package using TZ
CPack: Install projects
CPack: - Run preinstall target for: Demo8
CPack: - Install project: Demo8 [CPackSourceConfig.cmake]
CPack: Create package
CPack: - package: /c++/workspace/demo8/build/Demo8-1.0.1-Linux.tar.Z generated.


生成了三个不同格式的二进制文件包
[root@iZwz97bu0gr8vx4fbs6n1uZ build]# ls Demo8-*
Demo8-1.0.1-Linux.sh  Demo8-1.0.1-Linux.tar.gz  Demo8-1.0.1-Linux.tar.Z
这 3 个二进制包文件所包含的内容是完全相同的。可以执行其中一个。会出现一个由 CPack 自动生成的交互式安装界面：
[root@iZwz97bu0gr8vx4fbs6n1uZ build]# ./Demo8-1.0.1-Linux.sh
Demo8 Installer Version: 1.0.1, Copyright (c) Humanity
This is a self-extracting archive.
The archive will be extracted to: /c++/workspace/demo8/build

If you want to stop extracting, please press <ctrl-C>.
The MIT License (MIT)

Copyright (c) 2013 Joseph Pan(http://hahack.com)

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


Do you accept the license? [yn]:
y
By default the Demo8 will be installed in:
  "/c++/workspace/demo8/build/Demo8-1.0.1-Linux"
Do you want to include the subdirectory Demo8-1.0.1-Linux?
Saying no will install in: "/c++/workspace/demo8/build" [Yn]:
y

Using target directory: /c++/workspace/demo8/build/Demo8-1.0.1-Linux
Extracting, please wait...

Unpacking finished successfully
[root@iZwz97bu0gr8vx4fbs6n1uZ build]#

完成后提示安装到了 Demo8-1.0.1-Linux 子目录中，我们可以进去执行该程序：
[root@iZwz97bu0gr8vx4fbs6n1uZ build]# ./Demo8-1.0.1-Linux/bin/Demo 4 4
Now we use our own Math library.
4 ^ 4 is 256
[root@iZwz97bu0gr8vx4fbs6n1uZ build]#

将其他平台的项目迁移到 CMake
CMakeLists.txt 自动推导
gencmake 根据现有文件推导 CMakeLists.txt 文件。
CMakeListGenerator 应用一套文件和目录分析创建出完整的 CMakeLists.txt 文件。仅支持 Win32 平台。
```

[参考](https://www.hahack.com/codes/cmake/#USE-MYMATH-%E4%B8%BA-ON)
[github](https://github.com/wzpan/cmake-demo)





