<!--
 * @Author: csy-pc csydlut@163.com
 * @Date: 2022-11-23 14:35:43
 * @LastEditors: csy-pc csydlut@163.com
 * @LastEditTime: 2022-11-28 11:02:11
 * @FilePath: \arm-linux\Notes.md
-->
# 内容来源
跟我一起写Makefile（PDF重制版）

# Makefile语法
## 注意事项
**1. Makefile语法规定，命令列表中的每条命令必须以TAB键开始，禁止使用空格！**

**2. make命令会为Makefile中每个以tab开始的命令创建一个shell进程去执行。**

**3. 第一条规则的目标即为默认目标。**

**4. 变量都是字符串。**

**5. 注释文字要以 # 开始**

## make工作过程
1. make 会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文
件，并把这个文件作为最终的目标文件。
3. 如果edit 文件不存在，或是edit 所依赖的后面的.o 文件的文件修改时间要比edit 这个文件新，
那么，他就会执行后面所定义的命令来生成edit 这个文件。
4. 如果edit 所依赖的.o 文件也不存在，那么make 会在当前文件中找目标为.o 文件的依赖性，如
果找到则再根据那一个规则生成.o 文件。（这有点像一个堆栈的过程）
5. 当然，你的C 文件和H 文件是存在的啦，于是make 会生成.o 文件，然后再用.o 文件生成make
的终极任务，也就是执行文件edit 了。

这就是整个make 的依赖性，make 会一层又一层地去找文件的依赖关系，直到最终编译出第一个目
标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make 就会直接退出，并
报错，而对于所定义的命令的错误，或是编译不成功，make 根本不理。make 只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。
通过上述分析，我们知道，像clean 这种，没有被第一个目标文件直接或间接关联，那么它后面所定
义的命令将不会被自动执行，不过，我们可以显示要make 执行。即命令——make clean ，以此来清除
所有的目标文件，以便重编译。
于是在我们编程中，如果这个工程已被编译过了，当我们修改了其中一个源文件，比如file.c ，那
么根据我们的依赖性，我们的目标file.o 会被重编译（也就是在这个依性关系后面所定义的命令），于
是file.o 的文件也是最新的啦，于是file.o 的文件修改时间要比edit 要新，所以edit 也会被重新
链接了（详见edit 目标文件后定义的命令）。

## 规则格式

**prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。**

```
# Target可以是一个目标文件(object file)，也可以是一个可执行文件，或一个标签（Label）
# Prerequisites 生成Target所依赖的文件 和/或 目标
Target...: Prerequisites
    # 所执行的命令(任意一个shell命令)
    command1
    command2
```

```
main: main.o input.o calcu.o
    gcc -o main main.o input.o calcu.o
```
这条规则的目标是 main main.o、 input.o和 calcu.o是生成 main的依赖文件，如果要更新目标 main，就必须先更新它的所有依赖文件，如果依赖文件中的任 何一个有更新，那么目标也必须更新，“更新”就是执行一遍规则中的命令列表。

```
main: main.o input.o calcu.o
    gcc -o main main.o input.o calcu.o
main.o: main.c
    gcc -c main.c
input.o: input.c
    gcc -c input.c
calcu.o: calcu.c
    gcc -c calcu.c
```

上述代码中一共有 5条规则， 1~2行为第一条规则， 3~4行为第二条规则， 5~6行为第三条
规则， ，7~8行为第四条规则， 10~12为第五条规则， make命令在执行这个 Makefile的时候其执
行步骤如下：

首先更新第一条规则中的 main，第一条规则的目标成为默认目标，只要默认目标更新了那
么就认为 Makefile的工作。在第一次编译的时候由于 main还不存在，因此第一条规则会执行，
第一条规则依赖于文件 main.o、 input.o和 calcu.o这个三个 .o文件，这三个 .o文件目前还都没
有，因此必须先更新这三个文件。 make会查找以这三个 .o文件为目标的规则并执行。

以 main.o为例，发现更新 main.o的是第二条规则，因此会执行第二条规则，第二条规则里面的命令为“ gcc –c main.c”，这行命令很熟悉了吧，就是不链接编译 main.c，生成 main.o，其它两个 .o文件同理。
最后一个规则目标是 clean，它没有依赖文件，因此会默认为依赖文件都是最新的，所以其对应
的命令不会执行，当我们想要执行 clean的话可以直接使用命令“ make clean”，执行以后就会删
除当前目录下所有的 .o文件以及 main，因此 clean的功能就是完成工程的清理，“ make clean
的执行过程如图 3.4.1.1所示：


make的执行过程：
1. make命令会在当前目录下查找以 Makefile(makefile其实也可以 )命名的文件。
2. 当找到 Makefile文件以后就会按照 Makefile中定义的规则去编译生成最终的目标文件。
3. 当发现目标文件不存在，或者目标所依赖的文件比目标文件新 (也就是最后修改时间比
目标文件晚 )的话就会执行后面的命令来更新目标。


make工具就是在 Makefile中一层一层的查找依赖关系，并执行相应的命令。编译出最终的可执行文件。

Makefile的好处就是“自动化编译”，一旦写好了 Makefile文件，以后只需要一个 make命令即可完成整个工程的编译，极大的提高了开发效率。

## Makefile变量
**变量都是字符串**
```
# Makefile变量示例
# 定义变量objects
objects = main.o input.o calcu.o
# 使用变量objects
main: $(objects)
    gcc -o main $(objects)
```

## Makefile赋值
1. 赋值符"="
```
name = csy
curname = $(name)
name = ChenShuyi
# 输出结果:ChenShuyi
# = 允许使用后面的变量赋值。
# 注意和 := 对比
```
2. 赋值符":="
```
name = csy
curname := $(name)
name = ChenShuyi
# 输出结果：csy
# 使用 := 前面的代码的赋值，忽略后面的赋值操作。
```

3. 赋值符"?="
```
# 如果变量curname前面没被赋值，则将变量赋值为ChenShuyi
curname ?= ChenShuyi
```
1. 变量追加"+="
```
objects = main.o input.o
objects += calcu.o
# 此时，objects = main.o input.o calcu.o
```
需要给前面已经定义好的变量添加一些字符串进

## Makefile模式规则


## Makefile自动化变量
| 自动化变量 | 描述 |
| :----: | :---- |
| $@ | 规则中的目标集合，在模式规则中，如果有多个目标的话，“$@” ，“$@”表示匹配模式中定义的目标集合。|
| $% | 当目标是函数库的时候表示规则中的目标成员名，如果目标不是函数库文件，那么其值为空。 |
| $< | 依赖文件集合中的第一个文件，如果依赖文件是以模式(即"%")定义的，那么"$<"就是符合模式的一系列的文件集合。 |
| $? | 所有比目标新的依赖目标集合，以空格分开。 |
| $^ | 所有依赖文件的集合，使用空格分开，如果在依赖文件中有多个重复的文件，"$^"会去除重复的依赖文件，值保留一份。 |
| $+ | 和"$^"类似，但是当依赖文件存在重复的话不会去除重复的依赖文件。 |
| $* | 这个变量表示目标模式中"%"及其之前的部分，如果目标是 test/a.test.c，目标模式是a.%.c，那么 "$*"就是test/a.test |

传统写法
```
main: main.o input.o calcu.o
    gcc -o main main.o input.o calcu.o

main.o: main.c
    gcc -c main.c

input.o: input.c
    gcc -c input.c

calcu.o: calcu.c
    gcc -c calcu.c

clean:
    rm *.o
    rm main
```

使用自动化变量的写法
```
objects = main.o input.o calcu.o
main: $(objects)
    gcc -o main $(objects)

%.o : %.c
    gcc -c $<

clean:
    rm *.o
    rm main
```

## Makefile伪目标


## Makefile条件判断


## Makefile函数使用

test