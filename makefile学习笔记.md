# makefile学习笔记

### makefile规则

> ```
> target ... : prerequisites ...     目标文件：依赖文件
>     recipe ...(一定以tab键开头)      	要执行的命令
> ```

### makefile中使用变量

再makefile开头定义，将重复的文件定义成一个变量

> ```
> objects = main.o kbd.o command.o display.o \
>      insert.o search.o files.o utils.o
> ```

就可以将

> ```
> edit : main.o kbd.o command.o display.o \
>         insert.o search.o files.o utils.o
>     cc -o edit main.o kbd.o command.o display.o \
>         insert.o search.o files.o utils.o
> ```

简化为

> ```
> edit : $(objects)
>     cc -o edit $(objects)
> ```

### 包含其他的makefile

格式为

> include <filename>...

include前可以有空格但绝对**不能是tab**

这里有a.mk,b.mk,c.mk,foo.make,$(bar)，其包含了bish，bash，可以表示为

> ```
> include foo.make *.mk $(bar)
> ```

*.mk表示为以mk为后缀的文件的总称

##                                                                                         书写规则

> ```
> foo.o: foo.c defs.h      
>     cc -c -g foo.c
> ```

### 通配符

makefile支持*，？，~。

1，~/test 表示在home目录下的test目录

 	~hchen/test表示hchen的宿主目录下的test目录等同于/home/hchen/test

2，*.c表示所有.c为后缀的文件  （如果文件名中有通配符 *等可以用转义字符\ 来代替）

> object=*.o

意为object的值就是*.o。如果要使object的值是.o文件名的集合可以用：

> ```
> objects := $(wildcard *.o)
> ```



> ```
> $(patsubst %.c,%.o,$(wildcard *.c))
> ```

这个语句的作用是：

- 搜索当前目录下所有 `.c` 文件。
- 将这些 `.c` 文件的文件名模式从 `%.c` 替换为 `%.o`。（%的意思是匹配零或若干字符）
- 返回一个新的文件列表，其中所有 `.c` 文件的扩展名都被替换为了 `.o`。

那么可编译并链接所有.c和.o文件的语句为

> ```
> objects := $(patsubst %.c,%.o,$(wildcard *.c))
> foo : $(objects)
>     cc -o foo $(objects)
> ```

`patsubst` 是 `pattern substitution` 的缩写，它是 Makefile 中的一个函数，用于将一个文件列表中的每个元素从一种模式替换为另一种模式

### 文件搜寻

特殊变量VPATH的功能是将文件路径告诉make，让make去寻找这个文件

如果没有指明这个变量make只会在当前目录寻找依赖文件和目标文件，指明后make会在当前目录找不见的情况下到指定的目录寻找文件

> VPATH = src:../headers

这个定义制定了src和../headers这两个目录，make按照这个顺序搜索

（小写）vpath是一个make的关键字，使用方法：

1. **vpath** **<pattern>** **<directories>**    为符合模式<pattern>的文件指定搜索目录<directories>
2. **vpath** **<pattern>**    清除符合模式<pattern>的文件的搜索目录
3. **vpath **    清除所有已被设置好了的文件搜索目录

<pattern>需要包含%**（`%` 是模式规则中的特殊字符，用于定义规则如何从目标匹配到依赖项，而 `*` 是一个普通的通配符，用于文件名的搜索和匹配，但不能用于模式规则的定义）**

且vpath有顺序，make会按照语句先后执行搜索

> ```
> vpath %.c foo:bar
> vpath %   blish
> ```

这个语句的意思是先在foo找.c结尾的文件，然后再bar中找.c文件，最后在blish找任何文件（%匹配任意字符串）

### 伪目标

伪目标是一个标签，通过显式地指明这个目标让其生效。伪目标不能和文件名重名

可以用.PHONY显式指明目标是一个伪目标

> ```
> .PHONY : clean
> clean :
>     rm *.o temp
> ```

也可以为伪目标指定依赖文件

> ```
> all : prog1 prog2 prog3
> .PHONY : all
> prog1 : prog1.o utils.o
>     cc -o prog1 prog1.o utils.o
> prog2 : prog2.o
>     cc -o prog2 prog2.o
> prog3 : prog3.o sort.o utils.o
>     cc -o prog3 prog3.o sort.o utils.o
> ```

这样可以输入一个make就可以执行很多文件

### 多目标

要使用到自动化变量￥@

> ```
> bigoutput littleoutput : text.g
>     generate text.g -$(subst output,,$@) > $@
> ```

等价于

> ```
> bigoutput : text.g
>     generate text.g -big > bigoutput
> littleoutput : text.g
>     generate text.g -little > littleoutput
> ```

##                                                                                书写命令

### 显示命令

用@再命令前，这个命令不会被make显示

make执行时带入参数-n或--just-print只显示命令不会执行命令，可以用来调试

带入-s或--silent或--quite是全面禁止命令显示

### 命令执行

如果要使上一条命令的结果应用在下一条命令时应使用逗号分隔两条命令而不能写在两行

> ```
> exec:
>     cd /home/hchen
>     pwd
> ```

执行时其中的cd没有作用，pwd会打印出当前目录的makefile目录

> ```
> exec:
>     cd /home/hchen; pwd
> ```

执行时pwd会打印出/home/hchen

### 命令出错

如果其中一个规则中某个命令出错，make会退出，但命令出错不代表就是错误的。

1.如果要忽略命令的出错可以在前面加`-`   （tab之后）

2.全局办法：给make加上`-i`或`--ignore-errors`

3.-k或--keep-going 意为如果某规则命令出错了，终止该规则的执行，但是继续执行其他规则

### 嵌套执行make

可以通过嵌套执行make将每个目录里的make联系起来

有个子目录叫subdir，目录下有makefile，来指明了这个目录下文件的编译规则，总控可写为

> ```
> subsystem:
>     cd subdir && $(MAKE)
> ```

等价于


> ```
> subsystem:
>     $(MAKE) -C subdir
> ```

都意为：进入subdir然后执行make

传递变量到下级makefile：expert <variable...>;

不让变量传递到下级makefile：unexpert <variable...>

> ```
> export variable = value
> ```

> ```
> variable = value
> export variable
> ```

> ```
> export variable := value
> ```

上面三个语句等价

> ```
> export variable += value
> ```

> ```
> variable += value
> export variable
> ```

上面两个语句等价

如果传递所有变量只用一个expert就ok

SHELL和MAKEFLAGS总会传到下层makefile

不传递的参数：-c,-f,-h,-o,-w



好用滴：-w或--print-directory 作用是输出目前的工作目录    执行make  -w时

> ```
> make: Entering directory `/home/hchen/gnu/make'.
> ```

完成make离开目录时：（如果参数里有-s或--no-print-directory时-w失效）

> ```
> make: Leaving directory `/home/hchen/gnu/make'
> ```

### 定义命令包

相同命令可以用命令包定义

> ```
> define run-yacc
> yacc $(firstword $^)
> mv y.tab.c $@
> endef
> ```

命令行第一行命令，进行yacc程序，$^是一个自动变量，代表规则中所有依赖文件的列表$(firstword $^)函数去除列表的第一个单词，即第一个依赖文件的名称

第二行命令，重命名yacc生成的源文件。y.tab.c是yacc默认生成的文件名，$@为自动变量，代表规则的目标文件

像使用变量一样使用此命令包

> ```
> foo.c : foo.y
>     $(run-yacc)
> ```

##                                            使用变量

### 变量的基础

变量名不应该含有 `：`，`#` `=`或是`空格`

变量在声明时应附初值，且要在前加`$`,最好用`（）`或`{}`，如果要使用真实的 `$` 字符，用 `$$` 来表示

> ```
> objects = program.o foo.o utils.o
> program : $(objects)
>     cc -o program $(objects)
> 
> $(objects) : defs.h
> ```

可见见变量可以用在目标，以来，命令和新变量中

### 变量中的变量

定义变量的值的方法有两种

1.使用等号`=`： `=` 左侧是变量，右侧是变量的值。右侧变量不一定是已定义好的值也可以使用后面定义的值

​	但要在定义时避免递归定义。如果变量中有函数，运行会很慢。

2.使用`：=`：

> ```
> x := foo
> y := $(x) bar
> x := later
> ```

等价于：

> ```
> y := foo bar
> x := later
> ```

但是这样前面的变量不能使用后面的变量，只能使用前面已定义好了的变量

> ```
> y := $(x) bar
> x := foo
> ```

y的值是bar 不是foo bar

定义一个变量的值为空格

> ```
> nullstring :=
> space := $(nullstring) # end of the line
> ```

nullstring是Empty变量，啥也没有

先用Empty变量标明变量的值开始了，后面用#来表示定义变量的终止

**注意：**如果我们定义dir：=/foo/bar    #directory to put the frobs in那么dir的值就是/foo/bar再加四个空格

操作符`？=`

> ```
> FOO ?= bar
> ```

等价于

> ```
> ifeq ($(origin FOO), undefined)
>     FOO = bar
> endif
> ```

所以？=的意思是如果FOO没有被定义过，那么FOO就是bar，如果FOO被定义过，那么这条语句什么也不做

### 变量高级用法

1.变量值的替换：替换变量中的共有的部分，其格式是 `$(var:a=b)` 或是 `${var:a=b}` 意为把变量var中所有以	a结尾的a替换为b

> ```
> foo := a.o b.o c.o
> bar := $(foo:.o=.c)
> ```

​	把$(foo)中所有以.o结尾全部替换为.c，所以bar的值就是a.c b.c c.c

2.把变量的值再当成变量

> ```
> x = y
> y = z
> a := $($(x))
> ```

$(x)为y，$($(x))就是$(y)，所以$(a)的值就是z

进阶:

> ```
> x = $(y)
> y = z
> z = Hello
> a := $($(x))
> ```

$($(x))就是$($(y)),$(y)值为z，所以a:=$(z)即为Hello

再进阶：

> ```
> x = variable1
> variable2 := Hello
> y = $(subst 1,2,$(x))
> z = y
> a := $($($(z)))
> ```

$($($(z)))就是$($(y))，再等于$($(subst 1,2,$(x))) 。$(x)是variable1，subst函数把“variable1”中的所有“1”替换成“2”，所以“variable1”变成 “variable2”，所以$(a)的值就是$(variable2)即为Hello

### 追加变量值

使用+=给变量追加值

> ```
> objects = main.o foo.o bar.o utils.o
> objects += another.o
> ```

$(ogjets)变成main.o foo.o bar.o utils.o another.o

### override指令

有变量通过make名两行进行参数设置的，makefile文件对这个变量的赋值会被忽略。再makefile对这类参数进行设置可以这样：

> ```
> override <variable>; = <value>;
> override <variable>; := <value>;
> ```

或者追加

> ```
> override <variable>; += <more text>;
> ```

多行变量定义，用define指令，可以在此之前用override

> ```
> override define foo
> bar
> endef
> ```

### 多行变量

define指令后面跟的是变量的名字，重启一行定义变量的值，以endef结束。如果用define定义的命令变量没有以tab开头，make就不会认为它是命令。完整的如何使用多行变量：

> VAR = Hello, World!
>
> define print_message
> 	echo "Message:"
> 	echo "First line: $(VAR)"
> 	echo "Second line: This is a multi-line message."
> endef
>
> all:
> 	$(print_message)

### 环境变量

make运行时的环境变量可以在开始运行时被载入到makefile中，但如果make指定了“-e”参数，那么，系统环境变量将**覆盖**Makefile中定义的变量。

所以如果我们在环境变量中设置了CFLAGS，那么就可以在所有的makefile中使用这个变量，如果Makefile中定义了CFLAGS，那么则会使用Makefile中的这个变量。

### 目标变量

我们可以为某个目标设置局部变量，被称为“Target-specific Variable”，可以和全局变量同名，不会影响规则链以外的全局变量的值

> ```
> <target ...> : <variable-assignment>;
> <target ...> : overide <variable-assignment>
> ```

例子：

> ```
> prog : CFLAGS = -g
> prog : prog.o foo.o bar.o
>     $(CC) $(CFLAGS) prog.o foo.o bar.o
> prog.o : prog.c
>     $(CC) $(CFLAGS) prog.c
> foo.o : foo.c
>     $(CC) $(CFLAGS) foo.c
> bar.o : bar.c
>     $(CC) $(CFLAGS) bar.c
> ```

在这个例子中$(CFLAGS)永远是-g，不论全局的$(CFLAGS)是什么

### 模式变量

可以给定一个模式，把变量定义再符合这种模式的所有目标上

> ```
> %.o : CFLAGS = -O
> ```

意思是给所有以.o结尾的目标定义目标变量

> CFLAGS = -wall -g
>
> %.o : CFLAGS = -o

在这个例子中`CFLAGS` 默认被设置为 `-Wall -g`，但是任何以 `.o` 结尾的目标都会覆盖这个设置，使用 `-O` 作为它们的编译标志

##                                   使用条件判断

### 示例

> ```makefile
> libs_for_gcc = -lgnu
> normal_libs =
> foo: $(objects)
> ifeq ($(CC),gcc)
>     $(CC) -o foo $(objects) $(libs_for_gcc)
> else
>     $(CC) -o foo $(objects) $(normal_libs)
> endif
> ```

`libs_for_gcc = -lgnu`：这行定义了一个变量 `libs_for_gcc`，其值为 `-lgnu`，表示当使用 GCC 编译器时需要链接的库。`-lgnu` 是一个链接器选项，用于链接 GNU 库

`normal_libs`：定义`normal_libs`但没有进行赋值

`foo: $(objects)`：这是一个规则，表示目标 `foo` 依赖于 `$(objects)` 变量中列出的所有对象文件

`ifeq ($(CC),gcc)`：这是一个条件判断，检查变量 `$(CC)` 的值是否等于 `gcc`

`$(CC) -o foo $(objects) $(libs_for_gcc)`：如果条件为真，就执行这个，用gcc将所有文件连接成叫做foo的文件，并且链接gnu库

`$(CC) -o foo $(objects) $(normal_libs)`：如果条件为假，就用gcc将所有文件连接成叫foo的文件但是不链接任何库

### 语法

条件表达式的语法为

> ```
> <conditional-directive>
> <text-if-true>
> endif
> ```

> ```
> <conditional-directive>
> <text-if-true>
> else
> <text-if-false>
> endif
> ```

关键字

1.ifeq

ifeq(<arg1>,<arg2>)    ifeq'<arg1>' '<arg>'      ifeq "<arg1>" '<arg2>'    ifeq '<arg1>' "<arg2>"

都用来比较arg1和arg2的值是否相同

2.ifneq

ifneq (<arg1>, <arg2>)   ifneq '<arg1>' '<arg2>'   ifneq "<arg1>" "<arg2>"    ifneq"<arg1>" '<arg2>' 

比较arg1和arg2的值是否相同，如果不同就为真

3.ifdef

 ifdef <variable-name>

如果变量<variable-name>的值非空，则为真，否则为假。ifdef只是测试一个变量是否有值，不会吧变量扩展到当前位置

> ```
> bar =
> foo = $(bar)
> ifdef foo
>     frobozz = yes
> else
>     frobozz = no
> endif
> ```

> ```
> foo =
> ifdef foo
>     frobozz = yes
> else
>     frobozz = no
> endif
> ```

第一个$(frobozz)的值为yes，第二个就是no了

4.ifndef

> ```
> ifndef <variable-name>
> ```

如果<variable-name>的值为空则为真。

<conditional-directive>可以有空格，但是不能以tab开头，#也能用







 