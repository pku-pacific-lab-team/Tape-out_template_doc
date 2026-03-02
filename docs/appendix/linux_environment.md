# Linux 环境与常用工具

本文档补充介绍流片流程中常见的 Linux 环境操作与基础工具使用。

## 1. Terminal

---

## 2. Vim (Gvim)

### 2.1 GVIM 配置

全局配置文件：`~/.vimrc`。

设置字体的命令 `:set guifont = Courier\   14`，如果你不知道可用的字体名字，使用下面的命令可以得到一个字体名字的列表：`:set guifont=*`。

### 2.2 复制粘贴

1. 进入 normal /正常模式（刚进入 vim 的默认模式），如果你在 insert 模式，按下若干次 Esc 可以进入 normal 模式。
2. 把光标移动到开始复制的位置。
3. 按下 v 来选择字符（光标移动到结束复制的位置）。
4. 按下 y 来复制。
5. 光标移动到想要粘贴的位置，按下 p 粘贴。

---

## 3. Makefile

!!! Bug "FIXME!!!"
    基本结构

### 3.1 赋值

在 Makefile 中，= 用于定义递归变量，这意味着变量的值在每次使用时都会重新计算。
这在你需要在变量的值中使用其他变量，并且希望这些变量在使用时才被计算时非常有用。

例如，假设你有两个变量 `VAR1` 和 `VAR2`，你希望 `VAR2` 的值依赖于 `VAR1` 的当前值。你可以这样定义这两个变量：
```
VAR1 = value1
VAR2 = $(VAR1)
```

然后，你可以在后续的代码中改变 `VAR1` 的值，`VAR2` 的值会随之改变：

在这个例子中，如果你打印 `VAR2` 的值，你会看到 `value2`，而不是 `value1`。这是因为 `VAR2` 的值在每次使用时都会重新计算，所以它总是等于 `VAR1` 的当前值。

如果你在这个例子中使用 `:=` 而不是 `=`，`VAR2` 的值在定义时就被计算出来，所以无论 `VAR1` 如何改变，`VAR2` 的值都不会改变。

### 3.2 函数

- `$(filter-out pattern…,text)`：从 text 中过滤掉与 pattern 匹配的单词。
- `$(wildcard pattern)`：返回与 pattern 匹配的所有文件名。注意，该函数不会递归寻找！
- `$(addprefix prefix, names…)`：在 names 的每一个单词前面添加 prefix。

---

## 4. TCL

目前大部分的EDA工具都支持 `TCL` (Tool Command Language) 脚本语言，主要使用于发布命令给一些交互程序如文本编辑器、调试器和 Shell 。
`TCL` 的常用语法简单易懂，一些常用的用法示例如下：

``` tcl
# print a string
puts "Hello World!"

# set a variable
set variableA 10
set {variable B} test
puts $variableA  # 10
puts ${variable B}  # test

# perform arithmetic calculations
set a 21
set b 10
set c [expr $a + $b]
puts "Line 1 - Value of c is $c\n"  # Line 1 - Value of c is 31
set d [expr $a - $b]
puts "Line 2 - Value of d is $d\n"  # Line 2 - Value of d is 11
set e [expr $a * $b]
puts "Line 3 - Value of e is $e\n"  # Line 3 - Value of e is 210
set f [expr $a / $b]
puts "Line 4 - Value of f is $f\n"  # Line 4 - Value of f is 2
set g [expr $a % $b]
puts "Line 5 - Value of g is $g\n"  # Line 5 - Value of g is 1
```

此外还有 for 循环和 if-else 分支语句也是常用的语法。
配合[网络教程](https://www.yiibai.com/tcl/tcl_basic_syntax.html)和 ChatGPT 可以应对在本文档中碰到的所有 `TCL` 语句。

---

## 5. RISC-V Toolchain

<a id="gdb"></a>
### 5.1 GDB

!!! tip "如何使用"
    有关如何使用 GDB，请参考 [RISC-V CPU 调试：11.5 调试程序](cpu_debug.md)。

GDB（GNU Debugger）是一个强大的调试工具，广泛用于调试C、C++、Fortran等程序。
它通过多种机制与正在运行的程序（或程序的二进制文件）进行交互，以提供调试功能，如设置断点、单步执行、查看内存内容、修改变量值等。

GDB支持**远程调试功能**，这意味着GDB可以在一台机器上运行，而调试目标在另一台机器上。
GDB通过与远程的"GDB服务器"通信来控制目标程序的执行。
这种模式通常用于**嵌入式系统**调试。

GDB 依赖于编译器生成的**调试信息**（如 .debug_* 段）来提供源代码级别的调试。
GDB 使用这些调试信息来将二进制代码与源代码行、变量、函数等关联起来。

在反汇编文件中，`.debug_info`、.`debug_abbrev`、`.debug_line`等字段是**调试信息段**，它们是用于存储程序的调试信息的部分。
这些调试信息通常是在编译时通过添加 `-g` 选项生成的，用于**支持调试器**（如GDB）进行源代码级别的调试。

在编译过程中，`.debug_*` 段被嵌入到可执行文件中，但这些段**不属于**可执行文件的运行时所需的部分。
当程序正常运行时，这些段**不会**加载到内存中。
当使用调试器（如 GDB）调试程序时，调试器会主动读取可执行文件中的 `.debug_*` 段。

当 RISC-V CPU 进入调试模式时，CPU 停止正常的程序执行，并进入一个特殊的状态，允许调试器（如 GDB 通过 JTAG 接口）来控制和检查 CPU 的状态。

当 GDB 通过 JTAG 接口发送命令控制 CPU 时，命令的传输速率取决于 **JTAG 的频率**，但这些命令的执行（如设置断点、单步执行）仍然是按 **CPU 的主频**进行的。
