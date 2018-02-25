## CMake
你或许听过好几种 Make 工具，例如 GNU Make(**可见linux下的make笔记**) ，QT 的 qmake ，微软的 MS nmake，BSD Make（pmake），Makepp，等等。这些 Make 工具遵循着不同的规范和标准，所执行的 Makefile 格式也千差万别。这样就带来了一个严峻的问题：如果软件想跨平台，必须要保证能够在不同平台编译。而如果使用上面的 Make 工具，就得为每一种标准写一次 Makefile ，这将是一件让人抓狂的工作。

**CMake就是针对上面问题所设计的工具**：它首先允许开发者编写一种平台无关的 **CMakeList.txt** 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 工程。从而做到“Write once, run everywhere”。显然，CMake 是一个比上述几种 make 更高级的编译配置工具。

CMake 拥有比直接编写 Makefile 更直观的语法，更易于编写和维护。CLion也依赖于CMake。


具体用法可以参照网上手册，这里介绍简单用法。

### CMakeList
通过 CMake 管理的项目主目录下会有一个 CMakeLists.txt ，其中会包括工程包含哪些子目录等内容。 而在每个子目录下，也会包含一个 CMakeLists.txt， 用来管理该子目录中相关内容的构建。

```
cmake_minimum_required(VERSION 3.1)  # CMake 版本要求
PROJECT(hello)                       # 项目名称

aux_source_directory(. PROGRAM_SOURCE)  # 将当前目录所有文件添加到变量 PROGRAM_SOURCE 中

add_executable(hello ${PROGRAM_SOURCE}) # 指定目标可执行文件 hello 的源代码文件为 PROGRAM_SOURCE

add_library(hello  ${PROGRAM_SOURCE}) # 打包成静态库(.a)方便其他地方调用

add_library(hello  SHARED ${PROGRAM_SOURCE}) # 打包成动态库(.so)方便其他地方调用

```

以上示例，**add_executable就是最后会生成一个hello的可执行文件，而这个可执行文件的源文件是后者**

#### 添加外部头文件查找目录
当我们用到外部的库的时候，我们便需要添加外部库的头文件所在目录作为头文件查找目录。在 CMakeLists 中添加以下代码即可

```
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../
    ${CMAKE_CURRENT_SOURCE_DIR}../include)
```

CMake 中通过空格或者换行区分多个变量，上面的示例便是添加了两个目录到头文件查找路径中。

#### 添加外部链接库
通过以下代码可以添加外部链接库查找目录, 其中 CMAKE_CURRENT_SOURCE_DIR 是**内置宏**， 表示当前 CMakeLists.txt 所在目录：

```
link_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/../../lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../lib)
```

通过以下代码可以添加链接的外部库，这里链接 libmylib1 和 libmylib2， 这里链接的库可以是静态库也可以是动态库：

```
link_libraries(mylib1
    mylib2)
```

如果是链接指定目录指定某个库，则可以用以下方式：

```
target_link_libraries(hello ../mylib1.a
    hello ../mylib2.so)
```

#### Set命令添加变量

可以用set命令添加变量

```
set(COMMON_OBJ A B C)

add_executable(Main ${COMMON_OBJ})
```
