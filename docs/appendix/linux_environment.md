# Linux 环境与常用工具

本文档补充介绍流片流程中常见的 Linux 环境操作与基础工具使用。

## 1. Terminal

Terminal 是 Linux 环境下最核心的交互入口。流片相关的编译、仿真、分析与自动化脚本，大多都通过命令行执行。

### 1.1 常用命令速查

- `pwd`：显示当前目录。
- `ls` / `ls -al`：列出目录内容（含隐藏文件与详细信息）。
- `mkdir -p <dir>`：创建目录（可递归创建父目录）。
- `cp -r <src> <dst>`：复制文件或目录。
- `mv <src> <dst>`：移动或重命名文件。
- `rm <file>`：删除文件。
- `rm -r <dir>`：删除目录。
- `which <cmd>`：查看命令实际路径。

!!! warning "删除操作注意"
    `rm` 命令不会进入回收站。执行 `rm -rf` 前请先用 `pwd` 和 `ls` 确认路径。

### 1.2 路径与文件操作

- `.` 表示当前目录，`..` 表示上一级目录，`~` 表示用户家目录。
- 绝对路径从根目录 `/` 开始；相对路径相对于当前目录。

```bash
# 查看当前工程位置
pwd

# 进入某个目录
cd /project/holi/common

# 返回上一级
cd ..
```

### 1.3 查看日志与搜索内容

在调试编译和仿真问题时，最常用的操作是查看日志和定位关键字。

```bash
# 查看文件开头/结尾
head -n 30 build.log
tail -n 30 build.log

# 持续跟踪最新日志输出
tail -f sim.log

# 递归搜索关键词（推荐）
rg "ERROR|FATAL|Timing" logs/
```

### 1.4 环境变量与配置生效

很多 EDA 工具依赖环境变量（例如 `PATH`、工具安装路径、License 路径）。

```bash
# 查看变量
echo $PATH

# 临时设置变量（仅当前 shell 有效）
export RISCV=/opt/riscv

# 让配置长期生效：写入 ~/.bashrc 或 ~/.zshrc 后
source ~/.bashrc
```

### 1.5 常用快捷技巧

- 使用 `Tab` 自动补全命令和路径。
- 使用 `history` 查看历史命令。
- 使用 `Ctrl + r` 反向搜索历史命令。
- 使用管道 `|` 组合命令，例如：

```bash
# 查找所有包含 Warning 的行，并统计数量
rg "Warning" logs/ | wc -l
```

---

## 2. Vim (Gvim)

### 2.1 Gvim 配置

全局配置文件：`~/.vimrc`。

设置字体的命令 `:set guifont = Courier\   14`，如果你不知道可用的字体名字，使用下面的命令可以得到一个字体名字的列表：`:set guifont=*`。

### 2.2 复制粘贴

1. 进入 normal /正常模式（刚进入 Vim 的默认模式），如果你在 insert 模式，按下若干次 Esc 可以进入 normal 模式。
2. 把光标移动到开始复制的位置。
3. 按下 v 来选择字符（光标移动到结束复制的位置）。
4. 按下 y 来复制。
5. 光标移动到想要粘贴的位置，按下 p 粘贴。

### 2.3 模式与基础移动

Vim 常用 4 种模式：

- **Normal 模式**：默认模式，用于移动光标、复制删除、执行命令。
- **Insert 模式**：输入文本。按 `i` 进入，按 `Esc` 返回 Normal 模式。
- **Visual 模式**：选中文本。按 `v`（字符）、`V`（整行）、`Ctrl+v`（列块）。
- **Command 模式**：执行 `:` 命令（如保存、退出、替换）。

常用移动命令：

- `h j k l`：左下上右移动。
- `w b e`：按单词前进/后退/到词尾。
- `0` / `^` / `$`：行首/首个非空字符/行尾。
- `gg` / `G`：跳到文件开头/结尾。
- `Ctrl+d` / `Ctrl+u`：向下/向上翻半页。

### 2.4 高频编辑操作

- `x`：删除当前字符。
- `dd` / `yy`：删除当前行 / 复制当前行。
- `p` / `P`：在光标后 / 前粘贴。
- `u` / `Ctrl+r`：撤销 / 重做。
- `diw` / `ciw`：删除 / 修改当前单词。
- `.`：重复上一次编辑操作（非常高效）。

### 2.5 查找与批量替换

```vim
/pattern              " 向下查找 pattern
n                     " 跳到下一个匹配
N                     " 跳到上一个匹配
:%s/old/new/g         " 全文替换
:%s/old/new/gc        " 全文替换（逐个确认）
```

### 2.6 多文件与窗口协作

- `:e <file>`：打开文件。
- `:ls` + `:b <n>`：查看并切换 buffer。
- `:sp <file>` / `:vsp <file>`：水平/垂直分屏打开文件。
- `Ctrl+w` 再按 `h/j/k/l`：在窗口间移动。
- `:tabnew <file>` / `:tabnext`：新建/切换标签页。

### 2.7 可视块编辑与宏

可视块编辑（列编辑）适合批量加前缀、注释或对齐：

1. 按 `Ctrl+v` 进入列选择。
2. 选中多行列区域。
3. 按 `I` 输入内容，按 `Esc` 后会批量作用于选中行。

宏可以录制重复操作：

```vim
qa        " 开始录制到寄存器 a
...       " 执行一组编辑动作
q         " 结束录制
@a        " 执行一次宏
10@a      " 执行 10 次宏
```

### 2.8 推荐 `.vimrc` 最小配置

```vim
set number
set relativenumber
set tabstop=4
set shiftwidth=4
set expandtab
set autoindent
set hlsearch
set incsearch
set ignorecase
set smartcase
set clipboard=unnamedplus
```

说明：

- `expandtab`：将 `Tab` 转为空格，便于团队统一格式。
- `ignorecase + smartcase`：默认忽略大小写，输入大写时自动区分大小写。
- `clipboard=unnamedplus`：系统剪贴板与 Vim 互通（取决于 Vim 构建选项）。

---

## 3. Makefile

Makefile 用于管理构建流程，能把“编译、仿真、清理、打包”等步骤标准化并自动化。

### 3.0 Makefile 基本结构

Makefile 由**目标 (target)**、**依赖 (prerequisites)** 和**命令 (recipe)** 组成：

```make
target: dep1 dep2
	@echo "run target"
```

- 当依赖更新时，`make` 会重新执行对应目标命令。
- recipe 行必须以 **Tab** 开头（不是空格）。

### 3.1 变量

Makefile 中常见变量赋值方式如下：

- `=`：递归展开，使用时再求值。
- `:=`：立即展开，定义时求值。
- `?=`：仅在变量未定义时赋值。
- `+=`：追加内容。

```make
A = hello
B = $(A) world
A = hi

C := $(A) there
A = hey

D ?= default
SRC += top.v
```

上例中，`B` 最终是 `hi world`，`C` 保持为定义时的 `hi there`。

### 3.2 自动变量

规则中常用的自动变量：

- `$@`：当前目标名。
- `$<`：第一个依赖。
- `$^`：所有依赖（去重）。

```make
%.o: %.c
	$(CC) -c $< -o $@

app: a.o b.o
	$(CC) $^ -o $@
```

### 3.3 常用函数

下面是最常用的一组函数：

- `$(patsubst %.c,%.o,$(SRC))`：按模式替换（最常用）。
- `$(wildcard src/*.c)`：按通配符获取文件（不递归）。
- `$(filter %.c,$(SRC))`：保留匹配项。
- `$(filter-out %util.c,$(SRC))`：过滤掉匹配项。
- `$(addprefix -I,$(INC_DIRS))`：批量添加前缀。
- `$(addsuffix .o,$(MODULES))`：批量添加后缀。
- `$(subst src,build,$(PATHS))`：纯文本替换（非模式匹配）。
- `$(dir $(FILE))` / `$(notdir $(FILE))`：取目录名 / 文件名。
- `$(basename $(FILE))` / `$(suffix $(FILE))`：取无后缀名 / 后缀名。
- `$(foreach m,$(MOD),build/$(m).o)`：遍历列表并展开。
- `$(shell date +%Y%m%d)`：执行 shell 命令并获取输出。
- `$(if $(filter sim,$(MODE)),sim.log,run.log)`：条件展开。
- `$(sort $(LIST))`：排序并去重。

```make
SRC := src/a.c src/b.c src/util.c
OBJ := $(patsubst %.c,%.o,$(SRC))

C_SRC := $(filter %.c,$(SRC))
NO_UTIL := $(filter-out %util.c,$(SRC))

INC_DIRS := include core/include
INC_FLAGS := $(addprefix -I,$(INC_DIRS))

PATHS := src/main.c src/lib.c
NEW_PATHS := $(subst src,build,$(PATHS))
```

使用注意：

- `wildcard` 不递归查找子目录。
- `shell` 每次展开都可能执行命令，不建议在大循环中频繁使用。
- 函数展开时机会受 `=` 与 `:=` 影响，调试变量时要特别注意。

### 3.4 伪目标与默认目标

伪目标用于执行动作而非生成文件，建议显式声明：

```make
.PHONY: all clean run

all: app

clean:
	rm -f app *.o
```

一般把 `all` 放在文件前部作为默认目标，执行 `make` 时会优先运行它。

### 3.5 调试与可维护性

- `@`：隐藏单行命令回显，日志更干净。
- `$(info ...)`：打印调试信息。
- `make -n`：仅打印将执行的命令，不真正执行。
- `make -j`：并行构建，加速编译。
- 统一变量命名与目录结构（如 `SRC_DIR`、`BUILD_DIR`），避免路径硬编码。

---

## 4. Tcl

目前大部分 EDA 工具都支持 `Tcl` (Tool Command Language)。常见用途是串联工具命令、读取配置、批量生成报告。

### 4.1 变量、列表、字典

```tcl
# 变量
set top "soc_top"
set freq_mhz 200

# 列表
set libs [list slow.lib fast.lib tt.lib]
puts [lindex $libs 0]   ;# slow.lib

# 字典
dict set cfg mode "Averaged"
dict set cfg corner "tt_0p8v_25c"
puts [dict get $cfg mode]
```

### 4.2 流程控制

```tcl
set mode "Averaged"

if {$mode eq "Averaged"} {
    puts "run averaged flow"
} elseif {$mode eq "VectorFree"} {
    puts "run vector-free flow"
} else {
    puts "unknown mode"
}

foreach corner {ss tt ff} {
    puts "analyze corner: $corner"
}

for {set i 0} {$i < 3} {incr i} {
    puts "loop index = $i"
}

set retry 0
while {$retry < 2} {
    incr retry
}

switch -- $mode {
    Averaged   {puts "avg"}
    VectorFree {puts "vf"}
    default    {puts "n/a"}
}
```

### 4.3 过程与参数

`proc` 用于封装可复用逻辑：

```tcl
proc report_mode {mode {verbose 0}} {
    puts "mode = $mode"
    if {$verbose} {
        puts "verbose enabled"
    }
    return 0
}

report_mode "Averaged" 1
```

### 4.4 字符串与正则

```tcl
set line "  Timing  ERROR: setup violation  "
set clean [string trim $line]

if {[regexp {ERROR: (.+)} $clean -> msg]} {
    puts "found error: $msg"
}

set fixed [regsub {setup violation} $clean {setup_violation}]
puts $fixed
```

### 4.5 文件 I/O

```tcl
set fin [open "pt.log" r]
set fout [open "summary.log" w]

while {[gets $fin line] >= 0} {
    if {[string match "*WNS*" $line]} {
        puts $fout $line
    }
}

close $fin
close $fout
```

配合 [Tcl 基础语法教程](https://www.yiibai.com/tcl/tcl_basic_syntax.html) 和 ChatGPT，通常可以覆盖本文档涉及的脚本编写需求。

---

## 5. RISC-V Toolchain

<a id="gdb"></a>
### 5.1 GDB

!!! tip "如何使用"
    有关如何使用 GDB，请参考 [RISC-V CPU 调试：4. 调试程序](../post_tapeout/testing/cpu_debugging.md#4-调试程序)。

GDB（GNU Debugger）是一个强大的调试工具，广泛用于调试 C、C++、Fortran 等程序。
它通过多种机制与正在运行的程序（或程序的二进制文件）进行交互，以提供调试功能，如设置断点、单步执行、查看内存内容、修改变量值等。

GDB 支持**远程调试功能**，这意味着 GDB 可以在一台机器上运行，而调试目标在另一台机器上。
GDB 通过与远程的 "GDB 服务器" 通信来控制目标程序的执行。
这种模式通常用于**嵌入式系统**调试。

GDB 依赖于编译器生成的**调试信息**（如 .debug_* 段）来提供源代码级别的调试。
GDB 使用这些调试信息来将二进制代码与源代码行、变量、函数等关联起来。

在反汇编文件中，`.debug_info`、.`debug_abbrev`、`.debug_line`等字段是**调试信息段**，它们是用于存储程序的调试信息的部分。
这些调试信息通常是在编译时通过添加 `-g` 选项生成的，用于**支持调试器**（如 GDB）进行源代码级别的调试。

在编译过程中，`.debug_*` 段被嵌入到可执行文件中，但这些段**不属于**可执行文件的运行时所需的部分。
当程序正常运行时，这些段**不会**加载到内存中。
当使用调试器（如 GDB）调试程序时，调试器会主动读取可执行文件中的 `.debug_*` 段。

当 RISC-V CPU 进入调试模式时，CPU 停止正常的程序执行，并进入一个特殊的状态，允许调试器（如 GDB 通过 JTAG 接口）来控制和检查 CPU 的状态。

当 GDB 通过 JTAG 接口发送命令控制 CPU 时，命令的传输速率取决于 **JTAG 的频率**，但这些命令的执行（如设置断点、单步执行）仍然是按 **CPU 的主频**进行的。
