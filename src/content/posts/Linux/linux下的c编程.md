---
title: Linux下的C编程(一)
pubDate: 2024-4-30 12:41:30
categories: ['Linux']
description: 'Linux下C编程基础、常用开发工具'
---
目录<br>
第1节Linux下C编程基础<br>
第2节常用开发工具<br> 
第3节进程控制系统调用<br> 
第4节线程控制系统调用<br> 
第5节文件系统系统调用<br> 

## 第1节Linux下C编程基础 

#### 在Linux下进行C编程
$$\qquad$$首先要选择编辑器，常用的编辑器为VIM；
$$\qquad$$然后要选择编译器，常用的是GNU C/C++编译器 GCC（GCC是一种开源的编译器，安装自带，兼容各个版本）；
$$\qquad$$接下来选择调试器，应用最广泛的调试器是GDB<br>
$$\qquad$$同时还可以利用程序维护工具进行程序维护，make是Linux下较常用的程序维护工具。
##### 在安装Linux操作系统时选上“程序开发”中的“开发工具”，就可以自动安装GCC/GDB。<br>
$$\qquad$$Linux内核中设置了一组用于实现各种系统功能的子程序，称为系统调用。
$$\qquad$$用户可以通过系统调用命令在自己的应用程序中调用它们，从某种角度来看，系统调用和普通的函数调用非常相似。
$$\qquad$$区别仅仅在于，系统调用由操作系统核心提供，运行于核心态；而普通的函数调用由函数库或用户自己提供，运行于用户态。<br>
$$\qquad$$随Linux核心还提供了一些C语言函数库，这些库对系统调用进行了一些包装和扩展，因为这些库函数与系统调用的关系非常紧密，所以习惯上把这些函数也称为系统调用。
![l1-3.png](https://cdn.jsdelivr.net/gh/xrhm114/pic/images/20251128135502316.png)

## 第2节常用开发工具 
### GCC简介
$$\qquad$$目前Linux下最常用的C语言编译器是GCC（GNU Compiler Collection），它是Linux平台编译器的事实标准。
$$\qquad$$GCC是GNU项目中符合ANSI C标准的编译系统，能够编译用C、C++和Object C等语言编写的程序。
$$\qquad$$GCC之所以被广泛采用，还因为它能支持各种不同的目标体系结构。目前，GCC支持的体系结构有四十余种，常见的有X86系列、ARM、PowerPC等。同时，GCC还能运行在不同的操作系统上，如Linux、Solaris、Windows等。
$$\qquad$$使用GCC编译程序时，编译过程可以被细分为四个阶段：
$$\qquad \quad$$（1）预处理（Pre-Processing）
$$\qquad \quad$$（2）编译（Compiling）
$$\qquad \quad$$（3）汇编（Assembling）
$$\qquad \quad$$（4）链接（Linking） 
$$\qquad$$在这四个阶段中可以设置选项分别生成扩展名分别为“.i”、“.s”、“.o”的文件，以及最终可执行文件，各扩展名文件含义如下：
$$\qquad \quad$$.c：最初的c源代码文件。
$$\qquad \quad$$.i：经过编译预处理的源代码。
$$\qquad \quad$$.s：汇编处理后的汇编代码。
$$\qquad \quad$$.o：编译后的目标文件，含有最终编译出的机器码，但它里面所引用的其他文件中函数的内存位置尚未定义
![l1-4.png](https://cdn.jsdelivr.net/gh/xrhm114/pic/images/20251128142953012.png)
下面以程序hello.c为例具体看一下GCC是如何完成以上四个步骤的，程序hello.c源代码如下所示。
```c
#include<stdio.h>
int main(void)
{
   printf("Hello World!/n");
   return 0;
}
```
具体过程如下：<br>
（1）预处理阶段
$$\qquad$$在该阶段，编译器将上述代码中的stdio.h编译进来。GCC首先调用cpp进行预处理，根据以字符#开头的命令修改原始的C程序。
$$\qquad$$如hello.c中的指令#include <stdio.h>告诉预处理器读系统头文件stdio.h的内容，并把它直接插入到程序文本中去，结果就得到经过编译预处理的源代码hello.i。
```shell
$ gcc -E  hello.c  -o  hello.i  
```
（2）编译阶段
$$\qquad$$GCC调用ccl检查代码的规范性，是否有语法错误等，以确定代码实际要做的工作，在检查无误后，把代码翻译成汇编语言，生成汇编处理后的汇编代码hello.s。这个阶段对应的GCC命令如下所示。
```shell
$ gcc -S  hello.i  -o  hello.s  
```
（3）汇编阶段
$$\qquad$$GCC调用as把编译阶段生成的hello.s文件转成编译后的目标文件hello.o，但hello.c中所引用的其他文件中函数（如printf）的内存位置尚未定义。这个阶段对应的GCC命令如下所示：
```shell
$ gcc  -c  hello.s  -o  hello.o  
```
（4）链接阶段
$$\qquad$$GCC调用ld将程序的目标文件与所需的所有附加的目标文件连接起来，最终生成可执行文件。如GCC找到hello.c所调用的函数printf函数库所在位置/user/lib，把函数的实现链接进来，生成最终的可执行文件hello，可以利用下面的命令完成。
```shell
$ gcc hello.o -o hello 
```
#### 简化方法：一条命令完成所有步骤
```shell
$ gcc hello.c -o hello 
```
### GCC的使用 
```shell
格式：gcc [选项|文件]…
```
（1）总体选项 
|选项|含义|
|:---:|:---:|
|-c|只编译不链接，生成对应源文件的目标文件".o"|
|-S|只编译不汇编，生成汇编代码|
|-E|预处理后即停止，不进行编译|
|-o file|指定输出文件为file，file可以是可执行文件、目标文件、汇编文件等|
|-v|显示编译器内部编译各过程的命令行信息和编译器的版本号|
|-I dir|在头文件的搜索路径列表中添加dir目录|
|-g|指示编译程序在目标代码中加入供调试程序gdb使用的附加信息|

（2）链接选项
|选项|含义|
|:---:|:---:|
|-static|在支持动态链接库系统中，强制使用静态链接库，对其他系统不起作用|
|-shared|生成一个共享目标文件，可以和其他目标文件连接产生可执行文件|
|-L dir|把指定的目录dir加到链接程序搜索库文件的路径表中|
|-library|链接时搜索由library命名的库|
|-fpic|编译器输出位置无关目标码，适用于共享库（shared library）|
|-fPIC|编译器输出位置无关目标码，适用于动态链接（dymamic linking）|

（3）警告选项
|选项|含义|
|:---:|:---:|
|-pedantic|打开完全服从 ANSI C标准所需的全部警告诊断|
|-pedantic-errors|该选项和"-pedantic"类似，但是显示错误而不是警告|
|-W|禁止所有警告信息|
|-Wall|允许发出gcc提供的所有有用的报警信息|
|-Werror|视警告为错误，出现任何警告即放弃编译|

示例：<br>
（1）编译当前目录下的文件helloworld.c。
```shell 
$ gcc  helloworld.c
```
$$\qquad$$该命令将helloworld.c文件预处理、汇编、编译并链接形成可执行文件。这里未指定输出文件，默认输出为a.out，a.out为可执行程序文件名。
（2）将当前目录下的文件helloworld.c编译成名为helloworld的可执行文件。
```shell 
$ gcc  –o  helloworld  helloworld.c
```
（3）将当前目录下的文件helloworld.c编译为汇编语言文件。
```shell 
$ gcc  –S  helloworld.c 
```
$$\qquad$$该命令生成helloworld.c的汇编文件helloworld.s，使用的是AT&T汇编。
（4）将文件testfun.c 和文件test.c 编译成目标文件test
$$\qquad$$方法1：
```shell 
$ gcc  testfun.c  test.c  -o  test
```
$$\qquad$$方法2：
```shell 
$ gcc  -c  testfun.c  //将testfun.c编译成testfun.o
$ gcc  -c  test.c   //将test.c编译成test.o
$ gcc  testfun.o  test.o  -o  test  //将testfun.o和test.o链接成test
```
（5）编译当前目录下的程序bad.c，同时查看编译过程中所有报警信息。
$$\qquad$$程序bad.c的源码如下所示:
```c
#include <stdio.h> 
int main (void) 
{ 
   printf ("Two plus two is %f\n", 4); 
   return 0; 
}
```
$$\qquad$$编译并运行该程序:
```shell
[useralocalhost ~]$ gcc -Wall bad.c -o bad
bad.c:在函数'main'中：
bad.c:4:警告：格式‘f'需要类型'double',但实参2的类型为'int'
```
$$\qquad$$例中，对整数值来说，正确的格式控制符应该是%d。如果不启用 -Wall，程序表面看起来编译正常，但是会产生不正确的结果，具体如下所示:
```shell
[useralocalhost ~]$ gcc bad.c -o bad
[useralocalhost ~]$ ./badTwo plus two is 0.000000
```
### 简单的C语言程序
#### 例1：编写程序将a、b、c三个字符压入堆栈，然后依次从堆栈中弹出三个字符并打印在屏幕上。
```c 
/* stack.c */
char stack[512];
int top = -1;
void push(char c)
{
    stack[++top] = c;
}
char pop(void)
{
    return stack[top--];
}
int is_empty(void)
{
    return top == -1;
   }
```
```c
/* main.c */
#include <stdio.h>
void push(char);
char pop(void);
int is_empty(void);
int main(void)
{
    push('a');
    push('b');
    push('c');      
    while(!is_empty())
        putchar(pop());
    putchar('\n');
    return 0;
}
```
$$\qquad$$将两文件编译链接成可执行文件main并运行
```shell
[userelocalhost ~]$ gcc main.c stack.c -o main
[userelocalhost ~]$ ./main
cba
```
#### 例2：编写程序将a、b、c三个字符压入堆栈，然后从堆栈中依次弹出三个字符并打印在屏幕上。注：利用头文件的形式。
```c
/* stack.h */
void push(char);
char pop(void);
int is_empty(void);
```
```c
/* main.c */
#include <stdio.h>
#include "stack.h"
int main(void)
{
    push('a');
    push('b');
    push('c');
    while(!is_empty())
       putchar(pop());
    putchar('\n');
    return 0;
}
```
$$\qquad$$编译成目标文件。
$$\qquad$$因stack.h和main.c在同一个目录下，则用如下命令编译并运行。
```shell 
[useralocalhost ~]$ gcc -c stack.c
[useralocalhost ~]$ gcc -c main.c
[useralocalhost ~]$ gcc -o main main.o stack.o
[useralocalhost ~]$ ./main
cba
```
$$\qquad$$如果stack.h不在当前目录，则要指定目录。如stack.h在目录/home/user下，则用如下命令编译并运行。
```shell 
[userqlocalhost ~]$ gcc -o main main.o stack.o -I/home/user
[userelocalhost ~]$ ./main
cba
```
#### 例3：编写程序实现两个数字的和、差、乘积。要求编写五个文件:
##### main.c调用其它函数，实现和、差、乘积<br>sum.c,实现sum()函数，求两个数的和<br> sub.c，实现sub()函数，求两个数的差<br> mul.c，实现mul()函数，求两个数的乘积<br>calc.h，声明函数sum()sub()mul()<br>
```c
// calc.h
#ifndef CALC_H
#define CALC_H

int sum(int a, int b);
int sub(int a, int b);
int mul(int a, int b);

#endif
```
```c
// sum.c
#include "calc.h"

int sum(int a, int b) {
    return a + b;
}
```
```c
// sub.c
#include "calc.h"

int sub(int a, int b) {
    return a - b;
}
```
```c
// mul.c
#include "calc.h"

int mul(int a, int b) {
    return a * b;
}
```
```c
// main.c
#include <stdio.h>
#include "calc.h"

int main() {
    int x, y;
    printf("Enter two integers: ");
    scanf("%d %d", &x, &y);

    printf("Sum: %d\n", sum(x, y));
    printf("Subtraction: %d\n", sub(x, y));
    printf("Multiplication: %d\n", mul(x, y));

    return 0;
}
```
```shell 
[useralocalhost ~]$ gcc main.c sum.c sub.c mul.c -o calculator
[useralocalhost ~]$ ./calculator
输入整数: 8 9
求和: 17
求差: -1
求积: 72
```
### 程序调试工具gdb
程序中的错误通常分为以下三类：<br>
（1）编译时错误 <br>
$$\qquad$$又称为语法错误，主要是程序代码中有不符合所用编程语言语法规则的错误，如使用未定义的变量，括号不成对等。<br>
（2）运行时错误<br> 
$$\qquad$$编译器检查不出这类错误，仍然可以生成可执行文件，但在运行时会出错而导致程序崩溃。如除数为0，死循环等。<br>
（3）逻辑错误和语义错误 <br>

gdb是Linux系统中一个功能强大的GNU调试程序，它可以调试C和C++程序，使程序开发者在程序运行时观察程序的内部结构和内存的使用情况。gdb提供如下功能：<br>
$$\qquad$$（1）运行程序，设置所有的能影响程序运行的参数和环境。
$$\qquad$$（2）控制程序在指定的条件下停止运行。 
$$\qquad$$（3）当程序停止时，可以检查程序的状态。
$$\qquad$$（4）修改程序的错误，并重新运行程序。
$$\qquad$$（5）动态监视程序中变量的值。 
$$\qquad$$（6）可以单步逐行执行代码，观察程序的运行状态。
$$\qquad$$（7）分析崩溃程序产生的core文件。

### 程序维护工具make 
1．make的工作机制<br>
$$\qquad$$make的主要功能:自动检测一个大型程序的哪一部分需要重新编译，然后发出命令重新编译。GNU的make的工作过程如下：
$$\qquad$$（1）依次读入每个makefile文件；
$$\qquad$$（2）初始化文件中的变量；
$$\qquad$$（3）推导隐式规则，并分析所有规则；
$$\qquad$$（4）为所有目标文件创建依赖关系链；
$$\qquad$$（5）根据依赖关系和时间数据，确定哪些文件需要重新生成；
$$\qquad$$（6）执行相应生成命令。

2．makefile文件<br>
$$\qquad$$要用make维护一个程序，必须创建一个makefile文件，makefile文件告诉make以何种方式编译源代码和链接程序。makefile有自己的书写格式、关键字、函数，像C语言有自己的格式、关键字和函数一样，makefile描述规则组成如下所示。
$$\qquad$$目标：依赖文件
```java 
[TAB] 命令
```

例：写一个简单的makefile，描述如何创建最终的可执行文件“edit”，此可执行文件依赖于8个C源文件和3个头文件。
```make
#sample Makefile
edit : main.o kbd.o command.o display.o insert.o search.o files.o utils.o
	gcc -o edit main.o kbd.o command.o display.o insert.o search.o  files.o\ 
utils.o
main.o : main.c defs.h
	gcc -c main.c
kbd.o : kbd.c defs.h command.h
	gcc -c kbd.c
command.o : command.c defs.h command.h
	gcc -c command.c
display.o : display.c defs.h buffer.h
	gcc -c display.c
insert.o : insert.c defs.h buffer.h
	gcc -c insert.c
search.o : search.c defs.h buffer.h
	gcc -c search.c
files.o : files.c defs.h buffer.h command.h
	gcc -c files.c
utils.o : utils.c defs.h
	gcc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o insert.o search.o  files.o utils.o
```

3．make命令<br>
$$\qquad$$格式：make  [选项]  [目标文件]
$$\qquad$$说明：目标文件就是makefile文件中定义的目标之一，如果省略目标文件，make就将生成makefile文件中定义的第一个目标。
|<div style="width:100px;">选项|含义|
|-------------|:---|
|-C DIR|在读取 makefile前，进入目录“DIR”，就是切换工作目录到"DIR"后执行make。存在多个"C"选项时，make最终工作目录是每一个目录是前一个目录的相对路径。例如："make-C1-C etc"等价于"make-C letc"。|
|-d|make 执行时输出所有的调试信息。包括：make认为哪些文件需要重建；哪些文件需要比较它们的最后修改时间、比较的结果；重建目标所要执行的命令；使用的隐含规则等。|
|-e|使用系统环境变量的定义覆盖makefile中的同名变量定义。|
|-f=FILE|指定"FILE"为make执行的makefile文件。|
|-i|执行过程中忽略重新生产文件的命令过程中出现的错误信息。|
|-I DIR|指定被包含 makefile文件的搜索目录。在 makefile中出现"include"另外一个文件时，将在"DIR"目录下搜索。多个"-I"指定目录时，搜索目录按照指定顺序进行。|
|-k|执行命令错误时不终止 make的执行，make尽最大可能执行所有的命令，直到出现致命错误才终止。|
|-o FILE|指定文件"FILE"不需要重建,即使相对于它的依赖已经过期。因此依赖于文件"FILE"的目标也不会被重建。|
|-r|取消所有 make内嵌的隐含规则。|
|-w|在make进入一个目录读取makefile之前显示工作目录。这个选项可以帮助我们调试makefile，跟踪定位错误。|
|-W FILE|设定文件"FILE"的时间戳为当前时间，但不改变文件实际最后修改时间。此选项主要是为实现对所有依赖于文件"FILE"的目标的强制重建。|

示例：<br>
（1）在当前目录下已编辑好main.c、kbd.c、command.c、display.c、insert.c、search.c、files.c、utils.c八个源文件和defs.h、command.h、buffer.h三个头文件，调用上例中的makefile文件，生成目标文件edit。<br>
```shell    
$ make edit 或 $ make
```
（2）调用上例中的makefile文件，生成目标文件edit，文件files.o不须重建。
```shell 
$ make –o files.o  edit
```
4．make变量
$$\qquad$$makefile中的变量就像一个环境变量，作用是可以用来保存文件名列表、编译选项列表、程序运行的选项参数列表、搜索源文件的目录列表、编译输出的目录列表等。
$$\qquad$$make变量名是大小写敏感的，变量“foo”、“Foo”和“FOO”指的是三个不同的变量。makefile传统做法是变量名全采用大写的方式，推荐的做法是在对于内部定义的一般变量（例如目标文件列表objects）使用小写方式，而对于一些参数列表（例如编译选项CFLAGS）采用大写方式，但这不是必须的。
$$\qquad$$makefile中的变量是用一个文本串定义的，这个文本串就是变量的值，定义格式如下:
```shell 
VARNAME=string
```
$$\qquad$$如果要引用变量的值，可用如下方法:
```shell 
${VARNAME}
```
$$\qquad$$make解释规则时，VARNAME在等式右端展开为定义它的字符串，变量一般都在makefile的头部定义。
#### 例：下面的例子，使用变量“OBJS”作为所有的.o文件列表的替代，使用变量“CC”作为命令gcc的替代。
```make 
OBJS = main.o kbd.o command.o display.o  insert.o search.o files.o utils.o 
CC=gcc
edit: ${OBJS}
	${CC} -o edit ${OBJS}
main.o: main.c defs.h
	${CC} -c main.c 
kbd.o : kbd.c defs.h command.h
	${CC} -c kbd.c
command.o : command.c defs.h command.h
	${CC} -c command.c
display.o : display.c defs.h buffer.h
    ${CC} -c display.c
insert.o : insert.c defs.h buffer.h
	${CC} -c insert.c
search.o : search.c defs.h buffer.h
	${CC} -c search.c
files.o : files.c defs.h buffer.h command.h
	${CC} -c files.c
utils.o : utils.c defs.h
	${CC} -c utils.c
clean :
	rm edit ${OBJS}
```

<center>makefile 部分自动变量及含义</center>

|<div style="width:60px;">变量名|说明含义|
|:----|:----|
|$@|代表规则中的目标文件名。|
|$%|规则的目标文件是一个静态库文件时，代表静态库的一个成员名。如果目标不是函数库文件，其值为空。|
|$<|规则的第一个依赖文件名。|
|$^|规则的所有依赖文件列表，使用空格分隔。|
|$?|所有比目标文件更新的依赖文件列表，空格分割。|
|$*|如果目标文件的后缀是make识别的，$*就是去掉目标文件的后缀，只在隐含规则中才有意义。|
|$+|和少^相同不过保留了依赖文件中重复出现的文件。此变量会在特殊的状况下被创建，比如将自变量传递给连接器时重复是有意义的。|

```make
main :main.o func1.o func2.o
        gcc $^ -o $@
main.o :main.c func1.h func2.h
        gcc -c $<
func1.o:func1.c func1.h
        gcc -c $<
func2.o :func2.c func2.h
        gcc-c $<
.PHONY:clean
clean :
        rm *.o main
```
5．隐含规则<br>
隐含规则”为make提供了重建目标文件的通用方法<br>
（1）编译C程序
$$\quad$$“.o”文件自动由“.c”文件生成。
（2）编译C++程序
$$\quad$$“.o”文件自动由“.cc”文件生成。
（3） 汇编和需要预处理的汇编程序
$$\quad$$“.s”文件是不需要预处理的汇编源文件，“.S”是需要预处理的汇编源文件；“.o”文件可自动由“.s”文件生成；“.s”文件可由“.S”生成。
（4）链接单一的object文件
$$\quad$$“file”文件自动由“file.o”生成，通过C编译器使用链接器,此规则仅适用由一个源文件可直接产生可执行文件的情况。

#### 例：上面例题中的makefile文件使用隐含规则，就可以简化为下面的形式。
```make
OBJS = main.o kbd.o command.o display.o  insert.o search.o  files.o  utils.o 
CC=gcc
edit: ${OBJS}
	${CC} -o edit ${OBJS}
main.o: main.c defs.h
kbd.o : kbd.c defs.h command.h
command.o : command.c defs.h command.h	
display.o : display.c defs.h buffer.h	
insert.o : insert.c defs.h buffer.h	
search.o : search.c defs.h buffer.h	
files.o : files.c defs.h buffer.h command.h	
utils.o : utils.c defs.h	
clean :
	rm edit ${OBJS}
```

步骤1：编辑C源程序，可采用vim或emacs编辑器，源代码如下所示，main函数调用mytool1_print、mytool2_print这两个函数。
```c
#include "mytool1.h" 
#include "mytool2.h"  
int main(int argc,char **argv) 
{ 
   mytool1_print("hello"); 
   mytool2_print("hello"); 
}
```
步骤2：在mytool1.h中定义mytool1.c的头文件。
```c
/* mytool1.h */ 
#ifndef  _MYTOOL_1_H 
#define  _MYTOOL_1_H  
void mytool1_print(char *print_str);  
#endif
```
步骤3：用mytool1.c实现一个简单的打印显示功能。
```c
/* mytool1.c */ 
#include "mytool1.h" 
#include <stdio.h>
void mytool1_print(char *print_str) 
{ 
   printf("This is mytool1 print %s\n",print_str); 
}
```
步骤4：在mytool2.h中定义mytool2.c头文件。
```c
/* mytool2.h */ 
#ifndef  _MYTOOL_2_H 
#define  _MYTOOL_2_H  
void mytool2_print(char *print_str);  
#endif
```
步骤5：mytool2.c实现的功能与mytool1.c相似。
```c
/* mytool2.c */ 
#include "mytool2.h"
#include <stdio.h>
void mytool2_print(char *print_str) 
{ 
   printf("This is mytool2 print %s\n",print_str); 
}
```
步骤6：使用makefile文件进行项目管理，makefile文件内容如下。
```make
main:main.o mytool1.o mytool2.o 
	gcc -o main main.o mytool1.o mytool2.o 
main.o:main.c mytool1.h mytool2.h 
	gcc -c main.c 
mytool1.o:mytool1.c mytool1.h 
	gcc -c mytool1.c 
mytool2.o:mytool2.c mytool2.h 
	gcc -c mytool2.c
```
步骤7：将源程序文件和makefile文件保存在Linux下的同一个文件夹下，然后运行make编译链接程序并运行，如下所示。
```shell
[user@localhost c]$ make
gcc -c mytooll.c
gcc -c mytool2.c
gcc -o main main.o mytooll.o mytool2.o
[user@localhost c]$ ./main
This is mytooll print hello
This is mytool2 print hello
```
#### 对于上面的例3，其makefile文件如下：
```make
calculator: main.o sum.o sub.o mul.o
	gcc -o calculator main.o sum.o sub.o mul.o
# 各源文件的编译规则(依赖项)
main.o: main.c calc.h
	gcc -c main.c
sum.o: sum.c calc.h
	gcc -c sum.c
sub.o: sub.c calc.h
	gcc -c sub.c
mul.o: mul.c calc.h
	gcc -c mul.c
# 清理命令
clean:
	rm calculator main.o sum.o sub.o mul.o
```
```shell
[user@localhost c]$ make calculator
gcc -c main.c
gcc -c sum.c
gcc -c sub.c
gcc -c mul.c
gcc -o calculator main.o sum.o sub.o mul.o
[user@localhost c]$ ./calculator
输入整数: 5 6
求和: 11
求差: -1
求积: 30
```
