# awk 详解

awk 是一种编程语言，用于在linux/unix下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。它支持用户自定义函数和动态正则表达式等先进功能，是linux/unix下的一个强大编程工具。它在命令行中使用，但更多是作为脚本来使用。awk有很多内建的功能，比如数组、函数等，这是它和C语言的相同之处，灵活性是awk最大的优势。


## 使用方法
```
awk [options] 'script' var=value file(s)
awk [options] -f scriptfile var=value file(s)
```

尽管操作可能会很复杂，但语法总是这样，其中 pattern 表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。

awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，awk抽取信息后，才能进行其他文本操作。完整的awk脚本通常用来格式化文本文件中的信息。

通常，awk是以文件的一行为处理单位的。awk每接收文件的一行，然后执行相应的命令，来处理文本。


## 调用awk

```
1.命令行方式
awk [-F  field-separator]  'commands'  input-file(s)
其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。

2.shell脚本方式
将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。
相当于shell脚本首行的：#!/bin/sh
可以换成：#!/bin/awk

3.将所有的awk命令插入一个单独文件，然后调用：
awk -f awk-script-file input-file(s)
其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。
```

## awk的工作原理

awk 'BEGIN{ commands } pattern{ commands } END{ commands }'

第一步：执行BEGIN{ commands }语句块中的语句；

第二步：从文件或标准输入(stdin)读取一行，然后执行pattern{ commands }语句块，它逐行扫描文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。

第三步：当读至输入流末尾时，执行END{ commands }语句块。

BEGIN语句块 在awk开始从输入流中读取行 之前 被执行，这是一个可选的语句块，比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中。

END语句块 在awk从输入流中读取完所有的行 之后 即被执行，比如打印所有行的分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块。

pattern语句块 中的通用命令是最重要的部分，它也是可选的。如果没有提供pattern语句块，则默认执行{ print }，即打印每一个读取到的行，awk读取的每一行都会执行该语句块。




## awk内置变量

awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。


ARGC               命令行参数个数

ARGV               命令行参数排列

ENVIRON            支持队列中系统环境变量的使用

FILENAME           awk浏览的文件名

FNR                浏览文件的记录数

FS                 设置输入域分隔符，等价于命令行 -F选项

NF                 浏览记录的域的个数

NR                 已读的记录数

OFS                输出域分隔符

ORS                输出记录分隔符

RS                 控制记录分隔符


此外,$0变量是指整条记录。$1表示当前行的第一个域,$2表示当前行的第二个域,......以此类推。


案例-补充：
```
[root@dev01 yeyz_shell]# cat awk_test.txt 
this is a test file 
this is a test file 
this is a test file 
this is a test file 
this is a test file 
[root@dev01 yeyz_shell]# cat awk_test.txt | awk '{print $1,$2}'
this is
this is
this is
this is
this is
```

其中 awk '{print $1,$2}'是指打印出这个文件的第一列和第二列。当我们不指定分隔符的时候，awk会默认按照空格来进行分割，当字符中间的空格有多个的时候，awk会将连续的空格理解为一个分隔符。

当我们使用awk '{print $0}'的时候，会将这一列的值全部打印出来。如果需要拼接字符串的话，只需要在print的后面添加你想要拼接的字符串即可，如下：

```
[root@dev01 yeyz_shell]# cat awk_test.txt | awk '{print $1,$2,$3,"dog"}'
this is a dog
this is a dog
this is a dog
this is a dog
this is a dog
这里需要注意的是，$1和$2的两边不能加引号。
```

如何在收尾添加相关字符

首先我们再次给出awk的使用方法：

awk [option] 'pattern{action}' file1,file2,...filen

上面的例子说明了当action为print的时候，我们的测试结果，那么关于pattert，我们可以做如下测试。

如果我们想在awk的前后添加相关的声明信息，可以通过下面的方式：

```
[root@dev01 yeyz_shell]# cat awk_test.txt | awk 'begin{print "this is awk result:"} {print $1,$2,$3,"dog"} end{print "awk end"}'
this is a dog
this is a dog
this is a dog
this is a dog
this is a dog
[root@dev01 yeyz_shell]# cat awk_test.txt | awk 'BEGIN{print "this is awk result:"} {print $1,$2,$3,"dog"} END{print "awk end"}'
this is awk result:
this is a dog
this is a dog
this is a dog
this is a dog
this is a dog
awk end
```
可以看到，当我们把pattern的模式设置问BEGIN或者END的时候，它就可以在我们输出文件的时候，添加文件的首尾字符串，需要注意的是，BEGIN和END不能写为begin或者end。


分隔符
再次给出awk的基本语法：

awk [option] 'pattern{action}' file1,file2,...filen

上面两个例子，分别给出了awk命令关于例子1中的分隔符，我们在这个例子中详细说说，话不多说，看例子：

```
[root@dev01 yeyz_shell]# cat awk_test.txt 
this#is#a#test##file 
this#is#a#test##file 
this#is#a#test##file 
this#is#a#test##file 
[root@dev01 yeyz_shell]# cat awk_test.txt | awk -F# '{print $1,$2}' 
this is
this is
this is
this is
```
可以看到，当我们使用awk -F#的时候，awk命令就是用#作为分割符号，来分割这个文件中的内容了。


内置变量和自定义变量

上面三个例子分别从option、pattern以及action三个方面对awk命令进行了一些介绍，接下来我们看看awk命令当中的有些内置变量，常用的内置变量有：

NR   行号，当前处理文本行的行号

NF   当前行的字段的个数

FNR  个文件分别计数的行号

FILENAME  文件名称

FS   输入字段分隔符

OFS  输出字段分隔符

ARGC以及ARGV   数组以及命令行参数的个数

下面分别对这些变量进行举例说明：

FS  和  OFS  输入输出字段分隔符

FS的作用：

```
[root@dev01 yeyz_shell]# cat awk_test.txt 
this#is#a#test##file 
this#is#a#test##file 
this#is#a#test##file 
this#is#a#test##file 
[root@dev01 yeyz_shell]# cat awk_test.txt | awk -v FS='#' '{print $2,$3}'
is a
is a
is a
is a
```

FS使用#作为分隔符，对于输入的字符串进行处理。

下面的例子是OFS使用-作为分隔符，输出文件中的内容：

```
[root@dev01 yeyz_shell]# cat awk_test2.txt 
this is a shell program
this is a shell program
this is a shell program
this is a shell program
[root@dev01 yeyz_shell]# cat awk_test2.txt | awk -v OFS='-' '{print $2,$3,$4}'
is-a-shell
is-a-shell
is-a-shell
is-a-shell
```

NR和NF   行数和每一行的列数

测试样例如下：
```
[root@dev01 yeyz_shell]# cat awk_test3.txt 
this is a test program
this is a shell test program
[root@dev01 yeyz_shell]# cat awk_test3.txt | awk '{print NR,NF}'
 
 ```
结果显示，第一行有五个单词，第二行有六个单词

FNR和NR的区别
```
[root@dev01 yeyz_shell]# cat awk_test3.txt 
this is a test program
this is a shell test program
[root@dev01 yeyz_shell]# cat awk_test4.txt 
this#is#a#test#program
this#is#a#shell#test#program
[root@dev01 yeyz_shell]# awk {'print NR,$0'} awk_test3.txt awk_test4.txt 
 this is a test program
 this is a shell test program
 this#is#a#test#program
 this#is#a#shell#test#program
[root@dev01 yeyz_shell]# awk {'print FNR,$0'} awk_test3.txt awk_test4.txt 
 this is a test program
 this is a shell test program
 this#is#a#test#program
 this#is#a#shell#test#program
```

从测试可知，FNR是把不同文件的行号进行了区分，而NR没有对文件的行号进行区分。

FILENAME  显示文件名称

```
[root@dev01 yeyz_shell]# awk {'print FILENAME,FNR,$0'} awk_test3.txt awk_test4.txt 
awk_test3.txt  this is a test program
awk_test3.txt  this is a shell test program
awk_test4.txt  this#is#a#test#program
awk_test4.txt  this#is#a#shell#test#program
```
这个还是比较好理解的。
ARGC和ARGV

其中ARGV是一个数组，数组包含下标，使用下标可以访问数组中的文件名称，如下：

```
[root@dev01 yeyz_shell]# awk 'BEGIN{print "aaa",ARGV[1]}' test1 test2
aaa test1

[root@dev01 yeyz_shell]# awk 'BEGIN{print "aaa",ARGV[2]}' test1 test2
aaa test2

[root@dev01 yeyz_shell]# awk 'BEGIN{print "aaa",ARGV[0],ARGV[1],ARGV[2]}' test1 test2
aaa awk test1 test2

[root@dev01 yeyz_shell]# awk 'BEGIN{print "aaa",ARGV[0],ARGV[1],ARGV[2],ARGC}' test1 test2
aaa awk test1 test2 
```

需要注意的是，ARGV[0]指的是awk这个命令，这一点是awk命令规定的，其他的参数都是值得是后面处理的文件的名称，ARGC指的是ARGV数组的值的个数，在本例子中，它的值是3。

自定义变量

以上就是awk的内置变量，如果我们要自定义自己想要的变量，可以通过下面的方式来进行定义：

```
[root@dev01 yeyz_shell]# awk -v var='yeyz' 'BEGIN{print var}'
yeyz
[root@dev01 yeyz_shell]# awk  'BEGIN{ var2="yyy" ; print var2}'
yyy
```
当变量写在{}外面的时候，需要使用-v参数，当变量写在{}里面的时候，不需要写-v参数，但是需要注意的是，二者都需要写上BEGIN这个模式。


## 小结:

如果需要从输入的数据文件夹中取出特定的文本行,主要的工具为 grep 程序.

sed 是处理简单字符串替换的主要工具.大部分 shell 脚本在使用 sed 时几乎都是用来做替换的操作.
“从最左边开始,扩展至最长”这个法则描述了匹配的文本在何处匹配以及匹配扩展到多长.在使用 sed,awk 或其他交互式文本编辑程序时,这个法则相当重要.

cut 命令用以剪下选定的字符范围或字段,join 则是用来结合记录中具有共同键值的字段的文件.

awk 多半用于简单的”单命令行程序”,当你想要只显示选定的字段,或是重新安排行内的字段顺序时,就是 awk 排上用场的时候了.由于 awk 还是编程语言,即使在尖端的程序里,他也能发挥强大的作用.

