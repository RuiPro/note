# 什么是CMake

在了解CMake之前，我们先来看看我们是如何使用编译器的。

> 假设我们使用GCC编译器，当我们只有一个CPP文件main.cpp时，编译是非常简单的。
>
> ```
> g++ -o app main.cpp
> ```
>
> 这条指令将生成可执行文件app。
>
> 如果我们想要将文件func.h、func01.cpp、func02.cpp(其中func.h包含了两个函数的声明，两个头文件分别包含两个函数的实现)制作成库文件时，也比较简单。
>
> ```
> g++ -c func01.cpp func02.cpp
> ```
>
> 这条指令将得到两个用于制作静态库的二进制文件：func01.o func02.o。然后我们使用ar打包工具来制作一个静态库：
>
> ```
> ar rcs libfunc.a func01.o func02.o
> ```
>
> 这样我们就能得到静态库libfunc.a，搭配上func.h就可以使用了。
>
> 如果我们需要制作动态库，步骤如下：
>
> ```
> g++ -c -fpic func01.cpp func02.cpp
> ```
>
> 这条指令将得到两个用于制作动态库的二进制文件：func01.o func02.o。然后我们使用GCC来生成动态库：
>
> ```
> gcc -shared -o libfunc.so func01.o func02.o
> ```
>
> 这样我们就能得到动态库libfunc.so，搭配上func.h就可以使用了。

是否有些复杂？你可能会说不复杂，因为这还是在源文件数量较少的情况下。

大项目：那如果我们的源文件数以千计、并且不同功能的源文件保存在不同的目录下，你还会觉得简单吗？

跨平台：又或是我们编写的项目是可以跨平台的，那我们把项目从Linux上转移到Windows上时，想要使用MCVS编译器或clang编译器，我们又该如何编译？

多语言：更甚至我们的项目有些是只能用C编译器编译的，有些只能使用C++编译器编译，还有一些人使用java，你该怎么办？

为了解决这些场景，MakeFile问世了。MakeFile使用类似脚本的语言来告诉编译器应该如何进行编译，比如编译和链接的顺序。MakeFile相当于项目的蓝图，只要有蓝图(MakeFile文件)和材料(源文件)，我们就可以使用make命令一键编译出一栋大楼(可执行文件)。MakeFile语法示例如下。

```
CC = gcc
 TARGET = prog
 OBJS = main.o add.o
 INCLUDE = -I./include
 $(TARGET):$(OBJS)
     $(CC) $(OBJS) -o $(TARGET)   
 %.o:%.c
     $(CC) $(INCLUDE) -c $^ -o $@                                                                             
     
 .PHONY:clean
 clean:
     rm *.o prog
```

MakeFile像是一个脚本，它告诉编译器需要编译哪些文件，编译成什么文件，以及编译顺序。比如把func.cpp编译成func.o需要func.cpp和func.h头文件，需要先把func.o编译出来再编译main.cpp...

MakeFile有一个优点，它记录了我们的源文件时间戳。假设我们有1000个源文件，但是我们只对其中的若干个源文件进行了修改。想要重新编译时，MakeFile只会对这若干个修改过的源文件进行汇编和编译形成新的.o文件，再链接成一个主程序或库文件。如果我们不使用MakeFile，你只能在修改时就记下修改了哪些文件，再对这些文件进行针对性编译，或者重新编译这1000个源文件，这显然是非常累且耗时的工作。

可是想要维护一张好蓝图并不简单。就像计算机还未图形化普及时的蓝图必须由工程师使用铅笔和尺规画出，编写和修改都很不容易。我们的MakeFile也是一样的，MakeFile的语法比较晦涩，不易编写和维护，你也不会想要添加一个文件就查一次MakeFile语法和往MakeFile里合适的位置添加指令吧？



这时候CMake问世了。**CMake不直接指挥编译器**，它只是生成MakeFile文件，正如CMake官网所述的，CMake 是一种用于管理源代码构建的工具而非编译器。我们将CMake文件转成MakeFile文件后，依然要使用MakeFile工具指挥编译器进行编译。这就好比我们使用AutoCAD进行工程绘图，然后打印出蓝图进行建造。相比我们直接写一个MakeFile文件来说，**CMake的语法更友好、简洁。CMake在大项目、跨平台和多语言的场景中表现很出色**，而且它是开源的。和CMake类似的工具还有qmake(QT IDE)、nmake(VisualStudio IDE)等。

CMake目前支持以下C++编译器

- AppleClang：Apple Clang for Xcode 版本 4.4+
- Clang：Clang 编译器版本 2.9+
- GNU：GNU 编译器版本 4.4+
- MSVC：Microsoft Visual Studio 版本 2010+
- SunPro：Oracle Solaris Studio 版本 12.4+
- Intel：英特尔编译器版本 12.1+

当然，这需要参考[CMake官网](cmake.org)。



# 使用CMake

首先需要下载并安装CMake。



## 编写CMakeList

CMake依赖一个CMakeLists.txt文件或者以cmake为扩展名的文件。这个CMakeLists.txt文件文件名称是严格区分大小写的。我们可以编写一个CMakeLists.txt文件来配置我们的项目。

编写CMakeLists.txt文件需要遵循CMake的语法。CMake配置文件语法不区分大小写。CMake不支持中文，包括中文路径，因此你需要检查你的项目路径中是否含有中文。此外，在实际开发中，我们也要避免在项目中使用中文(注释除外)，这会导致很多问题。

一个CMake配置文件由CMake支持的命令、注释、空格和换行符构成，其中命令又分为CMake的基本语法、命令和变量。CMake不需要我们在末尾添加符号以示命令结尾，而要求我们每一个命令都占单独的行，一行一个命令。



#### 项目名称

使用`project(项目名称 支持的语言列表)`来指定项目名称和支持的语言。如果不添加支持的语言，则默认支持所有语言。

```
# 项目名称是MyProject
project(MyProject)
# 项目名称是MyProject，支持的语言是C++(CXX)
project(MyProject CXX)
```



#### 创建目标文件

- `add_library(动态库名 SHARE 源文件列表)`:用源文件列表中的源文件生成lib动态库名.so共享库

- `add_library(静态库名 STATIC 源文件列表)`:用源文件列表中的源文件生成lib静态库名.a静态库

- `add_executable(可执行文件名 源文件列表)`:用源文件列表中的源文件生成可执行文件

源文件列表也是变量列表的一种，也要遵循变量列表的规则。

源文件可以不带后缀，比如main.cpp可以写成main，CMake会自动寻找所有文件名为main的文件。但不推荐这样写，会引起混淆。

源文件名称不是CMake语法中的一部分，因此它是区分大小写的。但仍不支持中文。

```
# 用func.h和func01.cpp生成动态库libfunc.so
add_library(func SHARE func.h func01.cpp)
# 用func.h和func01.cpp生成动态库libfunc.a
add_library(func STATIC func.h func01.cpp)
# 用main.cpp、func.h和func01.cpp生成可执行文件app
add_executable(app 
			"main.cpp" 
			"func.h" 
			"func01.cpp")
```

或者你可以使用一个`SRC_FILES`变量来保存源文件列表，那么`add_executable(app main.cpp func.h func01.cpp)`这条语句就可以这样写：

```
set(SRC_FILES main.cpp func.h func01.cpp)
add_executable(app ${SRC_FILES})
```



#### 添加子项目

有时我们会将一个大项目拆分成几个小项目来编写。比如我们需要构建一个客户端和服务端，或者是一个可执行文件和多个库文件。这些小项目的源代码往往都不一样而且不能混淆。我们可以在CMake中配置小项目来实现。通过在大项目中使用add_subdirectory(子项目的文件夹 子项目的输出目录 EXCLUDE_FROM_ALL可选参数)命令，我们可以在CMake中配置子项目。

add_subdirectory(子项目的文件夹 子项目的输出目录)中

- 子项目的文件夹是指在执行此CMake配置时，连带执行子项目的文件夹中的CMake配置文件。**子项目仍需要一个独立的配置文件**。
- 子项目的输出目录是一个可选参数，用于配置编译子项目时产生的临时文件和目标文件存放的位置



#### CMake注释

CMake使用`#`符号来实现单行注释，也可以用`#[[注释内容]]`来实现多行注释

```
# 这是CMake注释
#[[	
	注释
	内容
]]
```



#### CMake输出

CMake使用`message("输出的内容")`来在终端中对内容进行输出。输出的内容必须使用`""`包起来。

```
message("Hello CMake!")
```



#### 最低要求的CMake版本号 

使用`cmake_minimum_required(VERSION 版本号)`来指定最低兼容的CMake版本，低于此版本的CMake不能使用此项目文件

```
# 最低要求3.10版本的CMake
cmake_minimum_required(VERSION 3.10) 
```



#### CMake变量

CMake变量分为正常变量(Normal)和缓存变量(Cache)。正常变量由用户定义，缓存变量是CMake自带的，比如临时文件存放的路径，只能在CMake执行时确定。

- 正常变量可以和缓存变量同名，但在使用时，会优先选择正常变量。
- 正常变量有作用域，缓存变量是全局的。

> 关于作用域：在默认情况下，每一个的目录或者函数都有自己的作用域，正常变量只在其被定义的作用域内有效。除非在定义时指定作用域参数。

我们可以使用`set(变量名 变量值(列表) 作用域参数)`来定义一个正常变量，用`${变量名}`来使用变量。这和C/C++宏中的`#define`类似。

我们可以使用`set(变量名 变量值(列表) CACHE 类型 <docstring> [FORCE])`

当变量值是一个列表时，我们可以使用空格分隔各个元素，也可以显式地使用`;`号分隔。但在使用时，CMake会将列表中的各元素用`;`号连接形成一个变量。

变量列表中的各元素可以使用`"元素"`的形式括起来，也可以不括。但如果元素内存在空格，那么就必须括起来，否则会被认为是两个元素。

```bash
set (var a b c)
message ("value = ${var}")
输出: value = a;b;c

还有以下格式
set(myVar a b c)  		# myVar = "a;b;c"
set(myVar "a" "b" "c")  # myVar = "a;b;c"
set(myVar a;b;c)  		# myVar = "a;b;c"
set(myVar "a b c") 	 	# myVar = "a b c"
set(myVar a b;c)  		# myVar = "a;b;c"
set(myVar a "b c")  	# myVar = "a;b c"
```

当变量值缺省时，实际上是把变量变为未设置状态，相当于使用了`unset(变量名)`命令。

但在流控制命令中，比如IF语句中，我们直接使用变量名而不使用`${变量名}`。



#### 有用的宏

|      | `变量`                                                       | `解释`                                                       |
| :--: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|  1   | `CMAKE_RUNTIME_OUTPUT_DIRECTORY` `CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG`  `CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE` | **可执行文件**的总/debug/release的输出路径，在未指定构建模式时，会使用总输出路径，下同 |
|  2   | `CMAKE_ARCHIVE_OUTPUT_DIRECTORY` `CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG` `CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE` | **静态库**的总/debug/release的输出路径                       |
|  3   | `CMAKE_LIBRARY_OUTPUT_DIRECTORY` `CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG` `CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE` | **动态库**的总/debug/release的输出路径                       |
|  4   | `【旧的 应弃用】EXECUTABLE_OUTPUT_PATH`                      | 可执行文件的输出位置，新版应使用1，如果新版变量和旧版变量同时指定了值，则新版变量优先使用，下同 |
|  5   | `【旧的 应弃用】LIBRARY_OUTPUT_PATH`                         | 库文件的输出位置，新版应使用3                                |
|  6   | `CMAKE_SOURCE_DIR`                                           | 顶层`CMakeLists.txt` 文件所在的文件夹，如果此`CMakeLists.txt` 包含子项目，则在子项目中此宏的值都相同 |
|  7   | `CMAKE_CURRENT_SOURCE_DIR`                                   | 当前`CMakeLists.txt` 文件所在的文件夹。如果当前的`CMakeLists.txt` 就是顶层时，此宏的值和6相同；如果当前的`CMakeLists.txt` 是子项目，则为此子项目的文件夹 |
|  8   | `PROJECT_SOURCE_DIR`                                         | 最近一次使用过`project()`的`CMakeLists.txt`所在的文件夹。大部分情况下和7一样 |
|  9   | `CMAKE_DEBUG_POSTFIX` `CMAKE_RELEASE_POSTFIX`                | 设置debug和release版本库文件的后缀名                         |
|  10  | `set_target_properties(${TARGET_NAME} PROPERTIES DEBUG_POSTFIX "_d")` <br> `set_target_properties(${TARGET_NAME} PROPERTIES RELEASE_POSTFIX "_r")` | 分别设置了Debug版本和Release版本下可执行文件的后缀名         |
|  11  | `CMAKE_BUILD_TYPE`                                           | 指定编译器的构建模式。可以选：`Debug`调试模式、`Release`发布模式、`MinSizeRel`最小体积发布、`RelWithDebInfo`带调试信息发布。 |



```
# 设置不同构建模式下的文件输出目录，请注意，这些语句必须在add_executable()之前设置才能生效
# 可执行文件
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/output/app)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/output/debug/app)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}/output/release/app)
# 动态库文件
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/output/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/output/debug/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}/output/release/lib)
# 静态库文件
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/output/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/output/debug/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}/output/release/lib)
```

```
# 设置同时生成静态库和动态库，如果不这样做会提示重名
add_library(mylib_static STATIC ${SOURCES})
add_library(mylib_shared SHARED ${SOURCES})
set_target_properties(mylib_static PROPERTIES OUTPUT_NAME mylib)
set_target_properties(mylib_shared PROPERTIES OUTPUT_NAME mylib)
```



## 启动CMake

### CMake工作流程

在编写完成CMakeLists.txt之后，我们可以使用CMake进行构建。但CMake本身不是一个编译器，因此它需要委托编译器去帮我们构建。因此，CMake构建项目的步骤有三步：

1. 编写CMakeLists.txt
2. 使用CMake将CMakeLists.txt生成makefile或sln文件，这些文件是编译器依赖的，这样编译器可以使用这些项目文件进行构建
3. 进行编译构建。

### 生成编译依赖文件

在终端中输入cmake命令即可。不过他有两个重要命令需要我们指定

- -S参数：指定源代码所在的路径，一般是CMakeLists.txt的路。只有指定了CMakeLists.txt在哪，cmake才能开始构建。此参数如果不指定，则默认使用当前目录

  ```
  # 让cmake使用当前目录下上一级目录下的src目录里的CMakeLists.txt配置来构建项目
  cmake -S ../src
  ```
  
- -B参数：指定生成的编译依赖文件存放的路径

  此参数用于指定cmake在构建过程中产生的临时文件和目标文件存放的路径。如果此路径不存在，cmake会创建。

  如果不指定此参数，cmake会把产生的临时文件和目标文件存放在当前目录。

  ```
  # 让cmake把构建过程中产生的临时文件和目标文件存放在当前目录下上一级目录下的build目录里
  cmake -S ../src	-B ../build
  ```


这一步你还可以指定一些编译选项

比如指定debug或release版本可以附加这个参数

```
-D CMAKE_BUILD_TYPE=Release
-D CMAKE_BUILD_TYPE=Debug
```

指定生成器

常用的生成器有makefile、ninja、visual studio

你可以使用`cmake -G`来查看可支持的生成器，以及cmake选择的默认生成器(带*号标识)，也可以使用它来指定一个支持的生成器(需要已安装)

```
cmake -G Ninja
```



### 进行编译构建

```
cmake --build
```







# Linux升级cmake方法

## 一、首先卸载旧版CMake

```
yum remove cmake
```



## 二、安装需要的模块

```
# 安装需要的组件
yum install -y libxml2 libxml2-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel zstd libzstd-devel curl libcurl-devel libpng libpng-devel
```



## 三、下载CMake3.23.0 源代码

```
# 下载
wget https://github.com/Kitware/CMake/releases/download/v3.23.0/cmake-3.23.0.tar.gz
```



## 四、编译CMake

```
# 解压
tar -zxvf cmake-3.23.0.tar.gz
# 进入到解压目录
cd cmake-3.23.0
# 配置
./configure --prefix=/usr/local/cmake-3.23.0
# 配置CPU核心数
gmake -j2
# 安装
gmake install
```



## 五、配置环境

```
#设置环境变量
touch /etc/profile.d/cmake.sh
chmod 777 /etc/profile.d/cmake.sh 
echo -e '\nexport PATH=/usr/local/cmake-3.23.0/bin:$PATH\n' >> /etc/profile.d/cmake.sh
source /etc/profile.d/cmake.sh

#设置头文件
ln -sv /usr/local/gcc-11.2.0/include/c++/11.2.0 /usr/include/c++/11.2.0

# 设置库文件
# touch /etc/ld.so.conf.d/gcc.conf
# chmod 777 /etc/ld.so.conf.d/gcc.conf 
# echo -e "/usr/local/gcc-11.2.0/lib64" >> /etc/ld.so.conf.d/gcc.conf

# 加载动态连接库
# ldconfig -v
# ldconfig -p |grep gcc
```





默认使用yum install gcc安装出来的gcc版本是4.8.5，可以升级到8.3.1，需要执行：

```
yum install centos-release-scl
yum install devtoolset-8-gcc*
```

安装完成后新版的gcc与g++安装在/opt/rh/devtoolset-8目录下，可以把devtoolset-8里的gcc和g++链接到/usr/bin去：

```
mv /usr/bin/gcc /usr/bin/gcc-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc
mv /usr/bin/g++ /usr/bin/g++-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/g++ /usr/bin/g++
```



# CentOS升级gcc

```
yum install centos-release-scl
yum install devtoolset-11-gcc*
mv /usr/bin/gcc /usr/bin/gcc-4.8.5 
ln -s /opt/rh/devtoolset-11/root/bin/gcc /usr/bin/gcc
mv /usr/bin/g++ /usr/bin/g++-4.8.5
ln -s /opt/rh/devtoolset-11/root/bin/g++ /usr/bin/g++
```

```
mv /usr/bin/gcc /usr/bin/gcc-4.8.5 && ln -s /opt/rh/devtoolset-11/root/bin/gcc /usr/bin/gcc && mv /usr/bin/g++ /usr/bin/g++-4.8.5 && ln -s /opt/rh/devtoolset-11/root/bin/g++ /usr/bin/g++
```



或者：

```
要从GCC 4.8.5更新到GCC 11.3.0，您需要执行以下步骤：
1. 下载GCC 11.3.0源代码：https://ftp.gnu.org/gnu/gcc/gcc-11.3.0/
2. 解压缩源代码：tar -xzvf gcc-11.3.0.tar.gz
3. 进入源代码目录：cd gcc-11.3.0
4. 配置GCC：./configure --disable-multilib
5. 编译GCC：make
6. 安装GCC：make install
7. 更新环境变量：export PATH=/usr/local/gcc-11.3.0/bin:$PATH
8. 检查GCC版本：gcc --version
```



# 生成的可执行文件使用自定义的库

一般我们会将可执行文件和动态库放在一起或放在环境变量下。

但这样很乱。

你可以使用GCC的`-rpath`选项来让可执行文件使用指定目录下的库

固定格式：`-Wl`是必须的

```
-Wl,-rpath-link 库的相对或绝对路径
```

如果你想把库文件都放在可执行文件的路径的lib文件夹下：

```
-Wl,-rpath-link '$ORIGIN/lib'		// $ORIGIN在linux中是程序所在的路径
```

CMake可以这样写

```
set_target_properties(构建目标名称 PROPERTIES
    BUILD_WITH_INSTALL_RPATH TRUE
    INSTALL_RPATH "\$ORIGIN/libs"
)
```

`$ORIGIN`的`$`需要转义，且整个值需要使用双引号括起来
