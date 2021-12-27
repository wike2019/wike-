# linux grep命令详解 


grep （global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。用于过滤/搜索的特定字符。可使用正则表达式能配合多种命令使用，使用上十分灵活。
## 基本介绍

```
grep [-acinv] [--color=auto] '搜寻字符串' filename

选项与参数：
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行
--color=auto ：可以将找到的关键词部分加上颜色的显示

```
## 规则表达式
```
^    # 锚定行的开始 如：'^grep'匹配所有以grep开头的行。    
$    # 锚定行的结束 如：'grep$' 匹配所有以grep结尾的行。
.    # 匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。    
*    # 匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。    
.*   # 一起用代表任意字符。   
[]   # 匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。    
[^]  # 匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。    
\(..\)  # 标记匹配字符，如'\(love\)'，love被标记为1。    
\<      # 锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。    
\>      # 锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。    
x\{m\}  # 重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。    
x\{m,\}   # 重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。    
x\{m,n\}  # 重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。   
\w    # 匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。   
\W    # \w的反置形式，匹配一个或多个非单词字符，如点号句号等。   
\b    # 单词锁定符，如: '\bgrep\b'只匹配grep。
```

## grep命令常见用法

```
在文件中搜索一个单词，命令会返回一个包含 “match_pattern” 的文本行：

grep match_pattern file_name
grep "match_pattern" file_name
在多个文件中查找：

grep "match_pattern" file_1 file_2 file_3 ...
输出除之外的所有行 -v 选项：

grep -v "match_pattern" file_name
标记匹配颜色 --color=auto 选项：

grep "match_pattern" file_name --color=auto
使用正则表达式 -E 选项：

grep -E "[1-9]+"
# 或
egrep "[1-9]+"
使用正则表达式 -P 选项：

grep -P "(\d{3}\-){2}\d{4}" file_name
只输出文件中匹配到的部分 -o 选项：

echo this is a test line. | grep -o -E "[a-z]+\."
line.

echo this is a test line. | egrep -o "[a-z]+\."
line.
统计文件或者文本中包含匹配字符串的行数 -c 选项：

grep -c "text" file_name
输出包含匹配字符串的行数 -n 选项：

grep "text" -n file_name
# 或
cat file_name | grep "text" -n

#多个文件
grep "text" -n file_1 file_2
打印样式匹配所位于的字符或字节偏移：

echo gun is not unix | grep -b -o "not"
7:not
#一行中字符串的字符便宜是从该行的第一个字符开始计算，起始值为0。选项  **-b -o**  一般总是配合使用。
搜索多个文件并查找匹配文本在哪些文件中：

grep -l "text" file1 file2 file3...
grep递归搜索文件
在多级目录中对文本进行递归搜索：

grep "text" . -r -n
# .表示当前目录。
忽略匹配样式中的字符大小写：

echo "hello world" | grep -i "HELLO"
# hello
选项 -e 制动多个匹配样式：

echo this is a text line | grep -e "is" -e "line" -o
is
line

#也可以使用 **-f** 选项来匹配多个样式，在样式文件中逐行写出需要匹配的字符。
cat patfile
aaa
bbb

echo aaa bbb ccc ddd eee | grep -f patfile -o
在grep搜索结果中包括或者排除指定文件：

# 只在目录中所有的.php和.html文件中递归搜索字符"main()"
grep "main()" . -r --include *.{php,html}

# 在搜索结果中排除所有README文件
grep "main()" . -r --exclude "README"

# 在搜索结果中排除filelist文件列表里的文件
grep "main()" . -r --exclude-from filelist

使用0值字节后缀的grep与xargs：

# 测试文件：
echo "aaa" > file1
echo "bbb" > file2
echo "aaa" > file3

grep "aaa" file* -lZ | xargs -0 rm

# 执行后会删除file1和file3，grep输出用-Z选项来指定以0值字节作为终结符文件名（\0），xargs -0 读取输入并用0值字节终结符分隔文件名，然后删除匹配文件，-Z通常和-l结合使用。
grep静默输出：

grep -q "test" filename
# 不会输出任何信息，如果命令运行成功返回0，失败则返回非0值。一般用于条件测试。
打印出匹配文本之前或者之后的行：

# 显示匹配某个结果之后的3行，使用 -A 选项：
seq 10 | grep "5" -A 3
5
6
7
8

# 显示匹配某个结果之前的3行，使用 -B 选项：
seq 10 | grep "5" -B 3
2
3
4
5

# 显示匹配某个结果的前三行和后三行，使用 -C 选项：
seq 10 | grep "5" -C 3
2
3
4
5
6
7
8

# 如果匹配结果有多个，会用“--”作为各匹配结果之间的分隔符：
echo -e "a\nb\nc\na\nb\nc" | grep a -A 1
a
b
--
a
b
```


## 总结：

为了方便回顾，将grep常用的选项总结如下

--color=auto或者--color：表示对匹配到的文本高亮显示

-i：搜索时不区分大小写

-n：显示匹配结果所在行号

-c：统计匹配到的行数，注意，是匹配到的总行数，不是匹配到的次数

-o：只显示符合条件的字符串，但不是整行显示，每个符合条件的字符串单独显示一行

-v：反向查询

-w：匹配整个单词，即精确匹配

-Ax：在输出时包含结果所在行之后的指定行数

-Bx：在输出时包含结果所在行之前的指定行数

-Cx：在输出时包含结果所在行之前和之后的指定行数

-e：实现多个选项的匹配，逻辑or关系

-q：静默模式，需要与echo $?联合用

-P：表示使用兼容perl的正则引擎

-E：使用扩展正则表达式，而不是基本正则表达式