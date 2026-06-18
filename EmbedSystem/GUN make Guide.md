# GNU make Guide

## GCC 编译器 - C 编译工具链

## `make`指令

## makefile 文件

makefile 是 GCC 编译工具调用 `make` 命令执行 Recompile 重编译程序时，以什么方式获取源文件以及输出目标文件的说明文档 / `Database`，会在调用 make 时自动查看该文档内容。所以 Build 的关键构筑文档为 makefile 文档。

> **注：makefile 文件命名推荐为 Makefile(首推) 或者 makefile 。**

### makefile 内容组成

makefile 主要由五个部分的内容组成：

* Explicit rules 显式规则：
  * Target 目标：生成的目标文件；
  * Prerequisites 先决条件：生成目标文件必备的文件；
  * Recipe 执行清单：如何执行从源文件到目标文件的过程；
* Implicit rules 隐式规则：GCC 工具链默认的一些规则与常识，可以在显式规则的基础上省略一些内容；
* Variable definition 变量定义：定义一个文本字符串为一个变量，在后续的文本中可以替换使用。
* Directives 指令：指 `make`指令在读取 makefile 的过程中同时需要进行的指令；
* Comments 注释：`#`开头的单独一行为注释说明；

```makefile
target ... : prerequisites ...
	recipe
	...
	...
```

注意 recipe 内容需要缩进，通常是一个制表符 `tab`，整个 makefile 就是这样一个个的目标构筑规则 `Rule` 组成的，例如以下 makefile 就是官方示例中的一个例子：

```makefile
# 示例模板，简单的 makefile 文件
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o

main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit main.o kbd.o command.o display.o \
           insert.o search.o files.o utils.o
```

以上蓝色字体，`:`之前的都是 Targets，紧跟在`:`之后的则是 Prerequisites，下面则是对应的 recipe。

Rule 指明两件事情：

* 什么情况下，目标文件是过期的，需要被更新；

  > 目标文件不存在；依赖的文件存在更新；

* 怎么样更新目标文件；

  > 根据 Recipe 指明的方式；

>`\`与 C 语言规则一致，用于链接分行显示的两段字符串，等效为单行。

### makefile 语法

### Implicit rule 隐式规则

#### 变量



