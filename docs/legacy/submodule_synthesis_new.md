# 数字子系统的逻辑综合（重构版）

!!! tip "TLDR"
    1. 模板文件路径：`/work/home/limingxuan/common/SOC_CVA6/`
    2. 仿真脚本：`/work/home/limingxuan/common/SOC_CVA6/Makefile`
    3. 仿真命令：`b make genus`

## 1. 模板文件

我们使用 CVA6 作为逻辑综合模板示例，其文件夹路径为：

```
/work/home/limingxuan/common/SOC_CVA6/
```

该文件夹的结构为：

```
SOC_CVA6
├── src                                         # RTL Source Files
│   ├── macro
│   │   └── dcim_ip_bm
│   │       ├── dcim_ip_bm.lef                  # DCIM physical library
│   │       ├── dcim_ip_bm_tt_0p80v_25c.lib     # DCIM timing library
│   │       └── dcim_ip_bm_tt_0p80v_85c.lib     # DCIM timing library
│   ├── sram                                    # Compiler Generated SRAM
│   │   └── sram_sp_hde
│   │       ├── sram_sp_hde.v                   # SRAM Behavior Module
│   │       └── ...
│   └── ...
├── syn
│   ├── scripts
│   │   ├── genus_synthesis.tcl                 # main synthesis script
│   │   ├── init_syn_none_opt.tcl               # init synthesis script
│   │   ├── init_syn_standard_opt.tcl           # init synthesis script
│   │   ├── init_syn_extreme_opt.tcl            # init synthesis script
│   │   └── syn_mmmc.tcl                        # define MMMC constraints
│   └── Makefile
├── config
│   ├── constraints_soc.sdc                     # define timing constraints
│   ├── global_define.tcl                       # define global parameters
│   └── user_define.tcl                         # user-specific parameters
├── Makefile                                    # Top-level Makefile
├── ...
...                                             # Other Folders/Files
```

!!! warning "关于模板文件"
    该 DCIM 自检电路仅作为**逻辑综合**流程的模板文件，其中不包含用于 Cadence Innovus 的脚本。
    如果希望后续进行**物理设计**流程，请使用[数字子系统的物理设计](submodule_implementation_new.md)中的模板文件！

## 2. 逻辑综合流程

### 2.1 添加源文件

在 `src/filelist.f` 中添加你的子模块源文件。

如果在子模块中例化了 SRAM IP，或者其他的定制 IP，则不需要在 `src/filelist.f` 中添加源文件，但是需要在后续综合参数中添加 SRAM 实例以及宏单元的名称。

### 2.2 修改综合参数

根据注释修改 `config/user_define.tcl` 中的综合参数。
你需要定义的参数如下：

- `rm_core_top`：顶层模块的名称。
- `rm_clock_pin`：顶层模块的时钟端口名称。
- `rm_clock_period`：时钟周期，单位 ns。
- `sram_insts`：SRAM 实例的名称，请在 `src/sram` 文件夹中使用 `sram_compiler` 生成对应名称的 sram 实例。
- `macro_insts`：宏单元的名称，请将对应文件（`.v`，`.lib`，`.lef`）放在 `src/macro/<name>` 文件夹中，分别命名为 `<name>.v` `<name>_[ss|tt|ff]_*v_*c.lib` `<name>.lef`。
- `std_lib, cell_ext`：选择标准单元库的阈值电压。
- `proj(analysis_view,[setup|hold])`：选择时序分析的 PVT，学术片考虑 `tt` 的两个选项即可。
- `syn_opt_level`：综合优化等级，可以选择 `none, standard, extreme`，优化程度依次递增。

!!! tip "lib 文件命名"
    lib 文件命名规则为 `<name>_<process>_<voltage>_<tempurature>`，可以参考 sram 的 lib 文件。

!!! warning "SRAM 与 Macro 文件"
    请注意，SRAM 与 Macro 文件夹下的文件名**务必**与 `sram_insts` 与 `macro_insts` 中的名称**一致**！参考模板文件夹的命名规则。

!!! question "综合参数进阶"

    如果你想要进一步修改综合参数，可以参考 `syn/scripts/init_syn.tcl`。

<a id="修改时序约束"></a>
### 2.3 修改时序约束

每个子模块的时序约束都需要**自行编写** `sdc` 文件，可以参考 `config/constraints_soc.sdc`。

你需要定义的内容如下：

- `create_clock`：设置时钟信号，告诉工具哪些信号是时钟。
- `set_clock_uncertainty`：设置时钟不确定性，所有时钟都需要设置 `setup, hold` 的不确定性。
- `set_clock_groups`：设置时钟组，告诉工具哪些时钟是同步的。
- `set_input_delay`：设置输入延迟，告诉工具输入信号的延迟。
- `set_output_delay`：设置输出延迟，告诉工具输出信号的延迟。
- `set_false_path`：设置假路径，告诉工具哪些路径不需要做时序分析。

其中，`set_input_delay` 和 `set_output_delay` 需要知道与该子模块对接的其他模块的时序信息，在第一次综合时可以假定为经验值`60% clock_period`。

!!! question "虚拟时钟"
    时序约束中，虚拟时钟是一个很常见的概念。
    虚拟时钟是为了描述**输入输出的时序信息**而引入的，对于综合工具来说，它**不了解**所综合的**子模块之外**的任何信息，因此需要一个虚拟时钟来告诉工具输入输出的时序信息。

    大部分情况下，虚拟时钟和实际时钟是**同步**的。
    实际上，这和直接将输入输出约束到实际时钟上是**等效**的，但是这样做会使得时钟树综合后的时序与预期不符，因此通常使用虚拟时钟约束 IO。

请将你编写的 `sdc` 文件命名为 `constraints_<top_module_name>.sdc`，并放在 `config/` 文件夹中。

### 2.4 运行逻辑综合

在 `SOC_CVA6` 文件夹下运行以下命令：

```
b make genus TOP=<top_module_name>
```

综合 CVA6 的时间大致需要 30-40 min 左右。
逻辑综合会在 `syn` 文件夹下生成 2 个文件夹 `logs, <top_module_name>`，文件结构如下所示。

```
syn
├── logs
│   ├── fv                              # empty functional verification folder
│   ├── <top_module_name>.cmd           # genus command file
│   └── <top_module_name>.log           # genus log file
├── <top_module_name>
│   ├── reports
│   │   ├── area
│   │   │   └── area.rpt                # area report
│   │   ├── timing
│   │   │   └── *_timing.rpt            # timing report
│   │   ├── power
│   │   │   ├── *_power_hier.rpt        # power hierarchy report
│   │   │   └── *_power.rpt             # power report
│   │   ├── time_intent
│   │   │   └── *_timing_intent_*.rpt   # timing constraint check
│   │   ├── runtime
│   │   │   └── check_*.rpt             # design check
│   │   └── others
│   │       └── *.rpt
│   ├── *.sdf                           # strandard delay format for post-synthesis simulation
│   ├── *_postsyn.v                     # generated netlist
│   └── design_backup                   # files for restoring design
│       └── ...
├── ...
...
```

### 2.5 查看主要输出报告

* `./syn/logs/<top_module_name>.log`：逻辑综合的日志文件，可以查找 `Error`, `Warning` 等关键词检查逻辑综合流程是否有误。
* `./syn/<top_module_name>/*_postsyn.v`：生成的门级网表，用于后续 Cadence Innovus 的后端设计
* `./syn/<top_module_name>/reports/timing/*_timing.rpt`：各个 PVT 的时序报告，可以查找 `VIOLATED` 关键词检查时序是否满足。
* `./syn/<top_module_name>/reports/area/area.rpt`：该模块的面积报告，可以作为后续后端设计版图大小的参考。

### 2.6 恢复设计

在进行一次逻辑综合后，可以在 `SOC_CVA6` 路径下通过如下指令快速恢复设计：

```
b make restore_genus TOP=<top_module_name>
```

!!! bug "待添加"

    关于脚本内容的解释。


!!! success ""
    特别感谢 [Wenjie Ren](https://ieeexplore.ieee.org/author/37089774513) 对本页内容的贡献和校对！
