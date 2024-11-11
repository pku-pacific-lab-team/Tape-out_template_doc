# 2(a). 数字子系统的逻辑综合（重构版）

!!! tip "TLDR"
    1. 模板文件路径：`/work/home/limingxuan/common/SOC_CVA6/`
    2. 仿真脚本：`/work/home/limingxuan/common/SOC_CVA6/Makefile`
    3. 仿真命令：`b make genus`

## 2.1 模板文件

我们使用开源 CPU CVA6 以及 AXI 总线构成的 SoC 作为逻辑综合模板示例，其文件夹路径为：

```
/work/home/limingxuan/common/SOC_CVA6/
```

该文件夹的结构为：

```
SOC_CVA6
├── src                             # RTL Source Files
│   └── ...
├── syn
│   ├── scripts
│   │   ├── genus_synthesis.tcl     # main synthesis script
│   │   ├── init_syn.tcl            # init synthesis script
│   │   ├── syn_mmmc.tcl            # define MMMC constraints
│   └── Makefile
├── utils
│   ├── constraints_soc.sdc         # define timing constraints
│   ├── global_define.tcl           # define global parameters
│   └── user_define.tcl             # user-specific parameters
├── Makefile                        # Top-level Makefile
├── ...
...                                 # Other Folders/Files
```

## 2.2 逻辑综合流程

### 添加源文件

在 `src/filelist.f` 中添加你的子模块源文件。

### 修改综合参数

根据注释修改 `utils/user_define.tcl` 中的综合参数。
你需要定义的参数如下：

- `rm_core_top`：顶层模块的名称。
- `rm_clock_pin`：顶层模块的时钟端口名称。
- `rm_clock_period`：时钟周期，单位 ns。
- `sram_insts`：SRAM 实例的名称，请在 `src/sram` 文件夹中使用 `sram_compiler` 生成对应名称的 sram 实例。
- `macro_libs, macro_lefs`：宏单元 LIB 文件和宏单元 LEF 文件的路径。
- `std_lib, cell_ext`：选择标准单元库的阈值电压。

!!! question "综合参数进阶"

    如果你想要进一步修改综合参数，可以参考 `syn/scripts/init_syn.tcl`。

### 修改时序约束

每个子模块的时序约束都需要**自行编写** `sdc` 文件，可以参考 `utils/constraints_soc.sdc`。

你需要定义的内容如下：

- `create_clock`：设置时钟信号，告诉工具哪些信号是时钟。
- `set_clock_uncertainty`：设置时钟不确定性，所有时钟都需要设置 `setup, hold` 的不确定性。
- `set_clock_groups`：设置时钟组，告诉工具哪些时钟是同步的。
- `set_input_delay`：设置输入延迟，告诉工具输入信号的延迟。
- `set_output_delay`：设置输出延迟，告诉工具输出信号的延迟。
- `set_false_path`：设置假路径，告诉工具哪些路径不需要做时序分析。

其中，`set_input_delay` 和 `set_output_delay` 需要知道与该子模块对接的其他模块的时序信息，在第一次综合时可以不考虑，但是在后续迭代综合时**务必添加**。

!!! question "虚拟时钟"
    时序约束中，虚拟时钟是一个很常见的概念。
    虚拟时钟是为了描述**输入输出的时序信息**而引入的，对于综合工具来说，它**不了解**所综合的**子模块之外**的任何信息，因此需要一个虚拟时钟来告诉工具输入输出的时序信息。

    大部分情况下，虚拟时钟和实际时钟是**同步**的。
    实际上，这和直接将输入输出约束到实际时钟上是**等效**的，但是这样做会使得时序约束文件变得复杂。

请将你编写的 `sdc` 文件命名为 `constraints_<top_module_name>.sdc`，并放在 `syn/scripts/` 文件夹中。

### 运行逻辑综合

在 `SOC_CVA6` 文件夹下运行以下命令：

```
b make genus
```

综合 SOC_CVA6 的时间大致需要 2.5h 左右。
逻辑综合会在 `syn` 文件夹下生成 3 个文件夹 `logs, *_data, *_reports`，文件结构如下所示。

```
syn
├── logs
│   ├── <top_module_name>.cmd       # genus command file
│   └── <top_module_name>.log       # genus log file
├── <top_module_name>_data
│   ├── *.sdf
│   ├── *.sdc
│   ├── *.mmmc.tcl
│   ├── *_postsyn.v                 # generated netlist
│   └── design_backup               # files for restoring design
│       └── ...
├── <top_module_name>_reports
│   ├── area
│   │   └── area.rpt                # area report
│   ├── timing
│   │   └── *_timing.rpt            # timing report
│   ├── power
│   │   ├── *_power_hier.rpt        # power hierarchy report
│   │   └── *_power.rpt             # power report
│   ├── time_intent
│   │   └── *_timing_intent_*.rpt   # timing constraint check
│   ├── runtime
│   │   └── check_*.rpt             # design check
│   └── others
│       └── *.rpt
├── ...
...
```

### 恢复设计

在进行一次逻辑综合后，可以在 `SOC_CVA6` 路径下通过如下指令快速恢复设计：

```
b make restore_genus
```

!!! bug "待添加"

    关于脚本内容的解释。