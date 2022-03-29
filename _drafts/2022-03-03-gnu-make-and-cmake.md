---
title: GNU Make和CMake
date: 2022-03-03 15:11:14 +0800
categories: [工具]
tags: [GNU Make, CMake]
toc: true
comments: true
math: true
pin: false
render_with_liquid: false
---

## GNU make

[Makefile Tutorial](https://makefiletutorial.com)

GNU（GNU's not Unix）的`make`工具能够自动确定大型程序的哪些部分需要重新编译，并发出命令来重新编译它们，通常用于编译C/C++程序（其他语言有类似的构建工具，脚本语言不需要构建工具），但不局限于编译程序。

使用`make`前，需要编写一个`Makefile`文件（通常放在项目目录中），在其中编写文件之间的依赖和更新文件的命令，之后在`Makefile`所在的目录下运行`make`。通常是通过执行较为复杂的编译命令来重新编译源代码文件，得到新的可执行文件。

一个简单的`Makefile`：

```makefile
hello:
	echo "hello world!"
```
{: file="Makefile"}

```text
echo "hello world!"
hello world!
```
{: file="shell output"}

在命令前面加`@`，终端不显示执行的命令：

```makefile
hello:
	@echo "hello world!"
```
{: file="Makefile"}

```text
hello world!
```
{: file="shell output"}

`Makefile`由多条规则（rule）组成，每条规则的格式为：

```makefile
targets: prerequisites
	commands
```

1. 目标（targets）是需要生成的文件的名称，用空格分开；通常一条规则只构建一个目标；也可以不生成任何文件，此时目标相当于给命令取的名称。
2. 依赖（prerequisites，dependencies）是生成目标文件需要参考的文件的名称，用空格分开，可以是其他目标；在构建目标之前需要保证所有依赖已经存在；任何一个依赖发生变化时，都需要重新构建目标。
3. 命令（commands）是构建目标的一系列步骤，以制表符开始，**制表符不能使用空格**，否则出现错误`Makefile:2: *** missing separator.  Stop.`。

如果目标没有被构建，或者目标的依赖有更新或没有被构建时，`make`才会重新运行构建命令。

每次都会运行：

```makefile
file1:
	echo "file 1"
```
{: file="Makefile"}

运行第一次后，如果不删除`file1`，不会再运行：

```makefile
file1:
	echo "file 1"
	touch file1
```
{: file="Makefile"}

```text
make: 'file1' is up to date.
```
{: file="shell output"}

添加新的依赖后，依赖还不存在，第一次会运行：

```makefile
file1: file2
	echo "file 1"
	touch file1

file2:
	echo "file 2"
	touch file2
```
{: file="Makefile"}

`all`目标通常用于构建除`clean`外的所有目标；`clean`目标通常用于删除其他目标输出文件；通常不放在默认目标的依赖中，而是在需要的时候运行`make clean`；`all`和`clean`都不是`make`中的关键字：

```makefile
all: say_hello generate

say_hello:
	@echo "hello world!"

generate:
	@echo "generating empty files"
	touch file-{1..13}.txt

clean:
	@echo "clean files"
	rm *.txt
```
{: file="Makefile"}

第一个目标是默认目标，运行`make`只会构建默认目标：

1. 可以在`Makefile`最开始用`.DEFAULT_GOAL`或`default_target`重新指定默认目标。
2. `.DEFAULT_GOAL`只能指定一个默认目标，`default_target`可以指定多个目标。
3. 如果同时使用了`.DEFAULT_GOAL`和`default_target`，只会将`.DEFAULT_GOAL`后的目标作为默认目标。
4. `.DEFAULT_GOAL`可以多次出现在任意位置，只有最后一次起作用。

```makefile
.DEFAULT_GOAL := generate
# default_target: generate say_hello

say_hello:
	@echo "hello world!"

generate:
	@echo "generating empty files"
	touch file-{1..13}.txt

clean:
	@echo "clean files"
	rm *.txt
```
{: file="Makefile"}

使用`:=`（或`=`）定义变量，使用`${}`（或`$()`）引用变量：

```makefile
CXX = g++

hello: hello.cpp
	${CXX} hello.cpp -o hello
```
{: file="Makefile"}

使用`.PHONY`声明伪目标，例如当执行`make clean`时，不会去检查`Makefile`所在目录是否有与`clean`同名的文件，而是直接构建`Makefile`中的`clean`目标；`.PHONY`可以多次出现在任意位置：

```makefile
.PHONY: all say_hello generate

all: say_hello generate

say_hello:
	@echo "hello world!"
	@echo "hello Eureka!\n"

generate:
	@echo "generating empty files"
	touch file-{1..13}.txt

clean:
	@echo "clean files"
	*.txt
.PHONY: clean
```
{: file="Makefile"}

通配符`*`可以用在目标、依赖和`wildcard`函数中，不能直接用于给变量赋值：

```makefile
thing_wrong := *.cpp    # Don't do this! '*' will not get expanded
thing_right := $(wildcard *.cpp)

all: one two three four

# Fails, because $(thing_wrong) is the string "*.o"
one: $(thing_wrong)

# Stays as *.o if there are no files that match this pattern :(
two: *.cpp

# Works as you would expect! In this case, it does nothing.
three: $(thing_right)

# Same as rule three
four: $(wildcard *.cpp)
```
{: file="Makefile"}

通配符`%`在匹配模式下匹配一个或多个字符，在替换模式下替换匹配的字符，通常用于规则的定义和特殊函数。

自动变量`$@`表示当前规则的第一个目标，`$^`表示当前规则的所有依赖，`$?`表示当前规则所有比目标新的依赖，

```makefile
file1 file2: file3 file4
	echo $@
	echo $?
	echo $^
```
{: file="Makefile"}

编译C程序时的隐式规则，不推荐使用，但已有很多人使用：

```makefile
CC = gcc # Flag for implicit rules
CFLAGS = -g # Flag for implicit rules. Turn on debug info

blah: blah.o
# Implicit rule #1: blah is built via the C linker implicit rule
# 	${CC} blah.o -o blah

blah.c:
	echo "int main() { return 0; }" > blah.c
# Implicit rule #2: blah.o is built via the C compilation implicit rule, because blah.c exists
# 	${CC} ${CFLAGS} -c -o blah.o blah.c

clean:
	rm -f blah*
```
{: file="Makefile"}

静态模式规则（static pattern rules），把目标中匹配目标模式的部分替换到依赖模式中，目标必须都匹配目标模式：

```makefile
targets: target-pattern: prereq-patterns
	commands
```
{: file="Makefile"}

```makefile
objects = foo.o bar.o all.o
all: $(objects)

# These files compile via implicit rules
# Syntax - targets ...: target-pattern: prereq-patterns ...
# In the case of the first target, foo.o, the target-pattern matches foo.o and sets the "stem" to be "foo".
# It then replaces the '%' in prereq-patterns with that stem
$(objects): %.o: %.c

all.c:
	echo "int main() { return 0; }" > all.c

%.c:
	touch $@

clean:
	rm -f *.c *.o all
```
{: file="Makefile"}

过滤器：

```makefile
obj_files = foo.result bar.o lose.o
src_files = foo.raw bar.c lose.c

.PHONY: all
all: $(obj_files)

$(filter %.o,$(obj_files)): %.o: %.c
	echo "target: $@ prereq: $<"
$(filter %.result,$(obj_files)): %.result: %.raw
	echo "target: $@ prereq: $<" 

%.c %.raw:
	touch $@

clean:
	rm -f $(src_files)
```
{: file="Makefile"}

```makefile
# Usage:
# make        # compile all binary
# make clean  # remove ALL binaries and objects

.PHONY = all clean

# 定义变量CXX为g++编译器
CC = gcc

LINKERFLAG = -lm

# 定义SRCS为所有以`.c`结尾的文件
SRCS := $(wildcard *.c)
# 定义BINS为SRCS中的所有文件去掉末尾的`.c`
BINS := $(SRCS:%.c=%)

all: ${BINS}

# %匹配所有目标，$<表示依赖，$@表示目标
%: %.o
	@echo "Checking.."
	${CC} ${LINKERFLAG} $< -o $@

# $<表示依赖
%.o: %.c
	@echo "Creating object.."
	${CC} -c $<

clean:
	@echo "Cleaning up..."
	rm -rvf *.o ${BINS}
```
{: file="Makefile"}

## CMake

CMake是一个用于构建、测试和打包软件的开源、跨平台工具系列，能够根据平台和编译器配置文件控制软件的编译过程，生成对应的`Makefile`文件。

```text
   _  __ __________   ______                                        
  | |/ //  _/ ____/  / ____/___ _____  ____ ___  ____  ______ _____ 
  |   / / // __/    / /_  / __ `/ __ \/ __ `/ / / / / / / __ `/ __ \
 /   |_/ // /___   / __/ / /_/ / / / / /_/ / /_/ / /_/ / /_/ / / / /
/_/|_/___/_____/  /_/    \__,_/_/ /_/\__, /\__, /\__,_/\__,_/_/ /_/ 
                                    /____//____/                    
```
