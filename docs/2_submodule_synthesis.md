# 数字子系统的逻辑综合

在数字芯片的设计流程中，后端设计是在逻辑综合的基础上进行的。在我们的模板文件中，逻辑综合和后端设计均会调用部分相同的脚本文件，因此先对数字子系统的逻辑综合流程做简要说明。

[逻辑综合](https://en.wikipedia.org/wiki/Logic_synthesis) (Logic Synthesis) 主要目的是将RTL级的设计进行优化，并映射到特定的工艺库中。此外，逻辑综合器也可以进行静态时序分析。

电路的逻辑综合一般由三个步骤组成：**转化、逻辑优化、映射**。在综合过程中，优化进程尝试完成标准单元的组合，使得组合能够最好满足设计的功能、时序和面积的要求。综合为约束驱动，给定的约束是优化目标。

## 逻辑综合的基本流程

数字子系统的逻辑综合使用`/work/home/ztzhu/tapeout_template/submodule/`文件夹。

> 若未做额外说明，**我们默认处于该文件夹路径下**。

逻辑综合的输入主要用到模板文件中的以下内容：

* `./rtl/`：包含数字子系统的RTL代码；
* `./scripts/`：Genus工具调用的脚本；
* `Makefile`：用于启动Cadence Genus工具；
* `./sram/`（_可选_）：数字子系统中例化的SRAM高速缓存或寄存器堆的专用IP核；
* `./asic_ip/`（_可选_）：数字子系统中例化的其他IP核（例如CiM Macro），或者更低层级的数字子系统。

### SRAM/Register File替换 _（可选）_

由于许多数字模块中依赖于较大规模的寄存器堆/SRAM高速缓存，这些模块需要替换成专门的IP核，而不是使用RTL代码直接综合，从而可以显著减小模块面积。

> 这一部分虽然不是必须的流程，但却可能造成较大的困惑，因此在此先进行说明。

主要用到的是ARM提供的`SRAM Compiler`和`Register File Compiler`（[工具路径](./1_index.md#arm-sram-compiler)）

在`./sram/`路径下新建文件夹，并在该文件夹下启动`SRAM Compiler`/`Register File Compiler`，用于存放生成的文件。
![sram compiler](figs/sram_compiler.png)

> 注意：`./sram/`路径下新建文件夹的名称与SRAM模块的`Instance Name`需保持一致！

#### SRAM Compiler使用说明

`SRAM Compiler`部分常用的设置选项如下：

* `Number of Words`: SRAM的深度。
* `Number of Bits`: SRAM的宽度。
* `Multiplexer Width`, `Number of Banks`会影响最终SRAM的形状，也受到数据深度与宽度的影响。在某些深度与宽度的组合下，可能无法找到一个合法的MUX与Bank数组合，在这种情况下可以考虑将SRAM的宽度减半，分开生成。
* `Frequency`保持与整体设计的时钟周期一致。
* `Bit Write Mask`允许你在写入数据时选择性地更新特定的位，而不用更新整个字（Word）。为此我们需要生成单独的掩码（Mask）信号来控制在每次写入SRAM时想要对哪几位进行操作。

在我们自己的数字子系统中使用SRAM Compiler生成的单元，需要生成相应的文件。

在`Corners`菜单中勾选所有的DOMAINS与PROCESSES，以保证生成综合报告的完整性。

![sram compiler corners](figs/corners.png)

在`views`部分选择`LEF Footprint`, `LVS Netlist`, `Liberty Model`, `Verilog Model`

* `Liberty Model`：用于逻辑综合与后端设计的时序分析与优化，包含SRAM的时序信息；
* `Verilog Model`：SRAM的Verilog代码，用于功能仿真；
* `LEF Footprint`：包含SRAM的版图信息（使用的金属层、IO位置等），为逻辑综合和后端设计提供SRAM的面积信息；
* `LVS Netlist`：用于后端设计的LVS检查，在逻辑综合阶段暂不需要。

![sram compiler views](figs/views.png)

#### Register File Compiler使用说明

与`SRAM Compiler`流程类似。
![register file compiler](figs/register_file_compiler.png)
