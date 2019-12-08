# 学习记录
## 安装(未测试)
```
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff5-dev libdc1394-22-dev         # 处理图像所需的包
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev liblapacke-dev
sudo apt-get install libxvidcore-dev libx264-dev         # 处理视频所需的包
sudo apt-get install libatlas-base-dev gfortran          # 优化opencv功能
sudo apt-get install ffmpeg

git clone https://github.com/opencv/opencv
cd opencv
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..
make 
make install (需要管理员权限)

sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig

编译的时候可以采用下面的命令
`pkg-config opencv --libs --cflags opencv`

```
## cmakelist 解释
参考：https://www.hahack.com/codes/cmake/
## tutorial 1
目标：
./Demo3
    +--- main.cc // 执行调用math中的函数进行计算
    +--- math/
          +--- MathFunctions.cc # 计算公式
          +--- MathFunctions.h
main文件夹下面CMakelist.txt
```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo3)
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
# 添加 math 子目录，这样会使math文件夹里面的cmakelist的内容也被执行
add_subdirectory(math)
# 指定生成目标 
add_executable(Demo main.cc)
# 添加链接库（可执行文件main会连接一个名为 MathFunctions 的链接库）
target_link_libraries(Demo MathFunctions)
```
math文件下CMakelist.txt
```
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)
# 生成链接库
add_library (MathFunctions ${DIR_LIB_SRCS})

```
### tutorial 2
任务：创建一个调用开关
例如，可以将 MathFunctions 库设为一个可选的库，如果该选项为 ON ，就使用该库定义的数学函数来进行运算。否则就调用标准库中的数学函数库。
```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo4)
# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  )
# 是否使用自己的 MathFunctions 库
option (USE_MYMATH
       "Use provided math implementation" ON)
# 是否加入 MathFunctions 库
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/math")
  add_subdirectory (math)  
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
# 指定生成目标
add_executable(Demo ${DIR_SRCS})
target_link_libraries (Demo  ${EXTRA_LIBS})
// 第7行的 configure_file 命令用于加入一个配置头文件 config.h ，这个文件由 CMake 从 config.h.in 生成，通过这样的机制，将可以通过预定义一些参数和变量来控制代码的生成
```
main.cpp
```
#include 
#include 
#include "config.h"
#ifdef USE_MYMATH
  #include "math/MathFunctions.h"
#else
  #include 
#endif
int main(int argc, char *argv[])
{
    if (argc < 3){
        printf("Usage: %s base exponent \n", argv[0]);
        return 1;
    }
    double base = atof(argv[1]);
    int exponent = atoi(argv[2]);
    
#ifdef USE_MYMATH
    printf("Now we use our own Math library. \n");
    double result = power(base, exponent);
#else
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#endif
    printf("%g ^ %d is %g\n", base, exponent, result);
    return 0;
}
```
config.h.in 这个文件会生成相应的config.h
```
#cmakedefine USE_MYMATH

```
运行
```
ccmake .
选择默认的USE_MYPATH
make .
```
可以看到结果，判断是否用了自己写的函数
### CMakelist里面的安装
首先先在 math/CMakeLists.txt 文件里添加下面两行：
```
//指定 MathFunctions 库的安装路径
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```
main文件夹下cmakelist
```
install (TARGETS Demo DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h"
         DESTINATION include)
```
通过上面的定制，生成的 Demo 文件和 MathFunctions 函数库 libMathFunctions.o 文件将会被复制到 /usr/local/bin 中，而 MathFunctions.h 和生成的 config.h 文件则会被复制到 /usr/local/include 中。
### CMakelist里面的测试文件
CMake 提供了一个称为 CTest 的测试工具。要在项目根目录的 CMakeLists 文件中调用add_test 命令。
```
# 启用测试
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
```
测试结果
```
[ehome@xman Demo5]$ make test
Running tests...
Test project /home/ehome/Documents/programming/C/power/Demo5
    Start 1: test_run
1/4 Test #1: test_run .........................   Passed    0.00 sec
    Start 2: test_5_2
2/4 Test #2: test_5_2 .........................   Passed    0.00 sec
    Start 3: test_10_5
100% tests passed, 0 tests failed out of 2
Total Test time (real) =   0.01 sec
```
#### 利用宏进行多项测试
```
# 定义一个宏，用来简化测试工作
macro (do_test arg1 arg2 result)
  add_test (test_${arg1}_${arg2} Demo ${arg1} ${arg2})
  set_tests_properties (test_${arg1}_${arg2}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)
 
# 使用该宏进行一系列的数据测试
do_test (5 2 "is 25")
do_test (10 5 "is 100000")
do_test (2 10 "is 1024")
```
### 支持gdb（GNU Project Debugger 的缩写, 是 Linux 下功能全面的调试工具）
开启-g模式
```
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```
### 添加环境检查
1. 添加 CheckFunctionExists 宏
首先在顶层 CMakeLists 文件中
```C++
# 检查系统是否支持 pow 函数
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake) //添加 CheckFunctionExists.cmake 宏
check_function_exists (pow HAVE_POW) //调用 check_function_exists 命令测试链接器是否能够在链接阶段找到 pow 函数
```
`将上面这段代码放在 configure_file 命令前`
2. 预定义相关宏变量
接下来修改 config.h.in 文件，预定义相关的宏变量。
```
#cmakedefine HAVE_POW
```
3. 在代码中使用宏和函数
最后一步是修改 main.cc ，在代码中使用宏和函数：
```
#ifdef HAVE_POW //如果已经有了pow函数
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#else //如果没有pow函数
    printf("Now we use our own Math library. \n");
    double result = power(base, exponent);
#endif
```
### 版本号
顶层 CMakeLists 文件，在 `project 命令之后`：
```
set (Demo_VERSION_MAJOR 1)\\主版本
set (Demo_VERSION_MINOR 0)\\副版本
```
#### 代码中获取版本信息
1. 修改 config.h.in 文件，添加两个预定义变量：
```
// the configured options and settings for Tutorial
#define Demo_VERSION_MAJOR @Demo_VERSION_MAJOR@
#define Demo_VERSION_MINOR @Demo_VERSION_MINOR@
```
2. main函数中调用Demo_VERSION_MAJOR
```
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include "config.h"
#include "math/MathFunctions.h"
int main(int argc, char *argv[])
{
    if (argc < 3){
        // print version info
        printf("%s Version %d.%d\n",
            argv[0],
            Demo_VERSION_MAJOR,
            Demo_VERSION_MINOR);
        printf("Usage: %s base exponent \n", argv[0]);
        return 1;
    }
    double base = atof(argv[1]);
    int exponent = atoi(argv[2]);
    
#if defined (HAVE_POW)
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#else
    printf("Now we use our own Math library. \n");
    double result = power(base, exponent);
#endif
    
    printf("%g ^ %d is %g\n", base, exponent, result);
    return 0;
}

```
### 生成安装包
配置生成各种平台上的安装包:二进制安装包和源码安装包。用到 CPack- CMake 提供的一个工具，专门用于打包。
1. 顶层的 CMakeLists.txt 文件`尾部`添加下面几行：
```
# 构建一个 CPack 安装包
include (InstallRequiredSystemLibraries) \\导入 InstallRequiredSystemLibraries 模块，以便之后导入 CPack 模块
set (CPACK_RESOURCE_FILE_LICENSE
  "${CMAKE_CURRENT_SOURCE_DIR}/License.txt") \\设置一些 CPack 相关变量
set (CPACK_PACKAGE_VERSION_MAJOR "${Demo_VERSION_MAJOR}") \\设置一些 CPack 相关变量
set (CPACK_PACKAGE_VERSION_MINOR "${Demo_VERSION_MINOR}") \\设置一些 CPack 相关变量
include (CPack)
```
2. 接下来的工作是像往常一样构建工程，并执行 cpack 命令。

生成二进制安装包：
```
cpack -C CPackConfig.cmake
```
生成源码安装包
```
cpack -C CPackSourceConfig.cmake
```


## 可视化图片
