# 数字子系统的物理设计

在完成数字模块的逻辑综合之后得到门级网表。以此为基础使用Cadence Innovus进行数字子系统的物理设计（后端设计）。

在后端设计中，许多脚本参数设置，例如电源网络的Width，Spacing，以及Macro的摆放位置需要根据版图大小和布局布线情况来设定，需要多次的迭代优化。

## 物理设计流程介绍

> 同[数字子系统的逻辑综合](./2_submodule_synthesis.md)，在后端设计中，我们继续使用`/work/home/ztzhu/tapeout_templates/submodule_tapeout/`文件夹。

物理设计的输入主要用到模板文件中的以下内容：

* `./work/`：Innovus的启动目录，存放Innovus的执行日志（包括用户输入命令和工具输出信息）；
* `./data/`：存放Genus逻辑综合得到的门级网表文件，也用于存放Innovus的输出文件（包括`GDSII`, `LEF`, `SDF`等） ；
* `./scripts/`：存放Innovus工具调用的脚本；
* `./my_scripts/`：存放用户进行物理设计各个阶段的命令脚本；
* `./sram/`（_可选_）：数字子系统中例化的SRAM高速缓存或寄存器堆的专用IP核（`CDL`, `LEF`, `LIB`, `Verilog`等）；
* `./asic_ip/`（_可选_）：数字子系统中例化的其他IP核（例如CIM Macro），或者更低层级的数字子系统的相关文件（`CDL`, `LEF`, `LIB`, `Verilog`等）。

该模板文件以一个**CVA6 RISC-V CPU**的后端设计流程作为一个示例。
在这个CPU中，包括用SRAM Compiler生成的Main Memory，I-Cache和D-Cache，以及定制设计的CIM模块。
我们按照顺序逐一介绍数字子模块物理设计的流程。

### 修改`init_invs.tcl`

在使用Innovus进行后端设计前，需要对工程脚本进行一些修改。

!!! tip "注意"
    在此，我们默认已经在逻辑综合之前修改了`./scripts/core_config.tcl`, `./scripts/design_inputs_macro.tcl`, `./scripts/tech.tcl`中部分内容（包括时钟周期、标准工艺库，添加了SRAM IP文件、ASIC IP文件等）。

在`./scripts/init_invs.tcl`中，定义了数字子系统的Power与Ground信号名称。

``` tcl
set init_gnd_net {VSS}
set init_pwr_net {VDD VDD_CIM}
```

在这个设计中，定制设计的CIM有单独的供电`VDD_CIM`，其余SRAM IP和标准单元共用`VDD`。

### 启动Cadence Innovus

打开一个终端，进入到`./work/`文件夹中，并启动Innovus。Innovus的日志文件位于`innovus.log`，用户通过终端命令行和GUI界面操作所对应执行的命令记录在`innovus.cmd`，均在`./work/`文件夹中。

``` shell
cd /work/home/ztzhu/tapeout_templates/submodule_tapeout/work/
rm -rf innovus.cmd* innovus.log* # clear previous innovus output files
b innovus
```

!!! tip "提示"
    每次启动innovus都会生成一个新的日志文件，所以我们建议经常清理，以免文件过多影响后续操作。

在终端中启动Innovus之后，可以在该终端中输入`TCL`脚本命令，也可以在弹出的Innovus GUI进行操作。
在我们的模板文件中主要依赖于命令行脚本进行后端设计。使用Innovus进行后端设计所用到的`TCL`命令均存放在`./my_scripts/`中。
使用文本编辑器打开`./my_scripts/innovus_script.tcl`（推荐使用GVIM）。

``` shell 
gvim /work/home/ztzhu/tapeout_templates/submodule_tapeout/my_scripts/innovus_script.tcl
```

观察`./my_scripts/innovus_script.tcl`，可以看到Innovus的物理设计流程大致分成了几个阶段，在Innovus终端中读入相应的命令即可（例如`source ../my_scripts/add_pin.tcl`）。
接下来按照顺序对物理设计流程中的关键命令进行说明。

### 执行`invs_init_setting.tcl`

观察`./my_scripts/invs_init_setting.tcl`中包含的命令。

#### 导入设计到Innovus

``` tcl
setMultiCpuUsage -localCpu 32
```

`setMultiCpuUsage`设置Innovus可以使用的CPU核数。对于现有的服务器，选择`-localCpu 32`可以保证Innovus稳定运行。

!!! Warning "警告"
    若使用`setMultiCpuUsage -localCpu max -cpuAutoAdjust true -verbose`，可能会导致Innovus闪退。


``` tcl
source -verbose ../scripts/core_config.tcl
source -verbose ../scripts/tech.tcl
source -verbose ../scripts/init_invs.tcl
source -verbose ../scripts/invs_setting.tcl
```

将此前修改过的4个脚本文件读入到Innovus中，包括数字子系统顶层模块名称（`$rm_core_top`）、各个文件的路径、标准工艺库的选择等内容。
此时我们经过逻辑综合的数字子系统已经导入到Innovus中。

``` tcl
saveDesign ${rm_core_top}.design_planning_init.enc
```

`saveDesign`命令在后端设计流程的各个步骤均会出现，用于保存目前后端设计的进度。

!!! tip "提示"
    在后续的物理设计流程中，若出现错误或者需要回溯到之前的设计进度，可以使用`restoreDesign`命令载入之前保存的设计。
    例如(bug here)：`restoreDesign ${rm_core_top}.design_planning_init.enc`

!!! bug

#### 设置后端不使用的标准单元

``` tcl
set_dont_use [get_lib_cellls FA1D0BWP7T30P140HVT]
set_dont_use [get_lib_cellls FA1D1BWP7T30P140HVT]
set_dont_use [get_lib_cellls FA1D2BWP7T30P140HVT]
```

上述三个全加器标准单元在后续DRC验证时会报错，因此在布局布线之前设置不使用这些标准单元。

#### 设置版图大小

``` tcl
set cell_height 0.7
set macro_halo_spc [expr 1 * $cell_height]
set macro_halo_spc_2 [expr 2 * $cell_height]
set macro_halo_spc_4 [expr 4 * $cell_height]
set die_sizex 1200
set die_sizey 1200
```

以上命令设置了几个物理设计中所使用到的变量大小：

* `$cell_height`是标准单元的高度；
* `$macro_halo_spc`用于设置Macro Route Blockage的宽度。
* `$die_sizex`, `$die_sizey`分别是该模块版图的物理宽度与物理高度。

!!! Bug
    Route Blockage vs. Halo

``` tcl
floorPlan -d $die_sizex $die_sizey 3.5 3.5 3.5 3.5
uiSetTool select
getIoFlowFlag
```

一个初始的版图类似下图所示。

<figure>
  <img src="../figs/example_floorplan.png" width=80%>
  <figcaption>Example Floorplan</figcaption>
</figure>

中央带有横条纹的区域称为Core box，用于摆放Macro和标准单元。所有标准单元的宽度各不相同，但是高度均为`$cell_height`（对于22nm工艺，即为0.7um）。
在Core box的四周到I/O管脚之间通常留有一定的间距，用于摆放Core Ring和I/O管脚布线。

`floorPlan`命令用于指定版图的大小。

`floorPlan -d {<W H Left Bottom Right Top>}`，使用该命令设置版图大小总共需要6个参数，分别代表：

* 完整版图 (Die box) 的宽度
* 完整版图 的高度
* Core box（用于摆放标准单元的版图部分）到I/O左侧边界的距离
* Core box到I/O底部边界的距离
* Core box到I/O右侧边界的距离
* Core box到I/O上方边界的距离

因此，设置上述版图大小的命令为：`floorPlan -d 100 100 10 10 10 10`。

### 执行`place_macro.tcl`

查看`./my_scripts/place_macro.tcl`中包含的命令。

#### 查看Macro

在Innovus GUI界面右上角选择`Floorplan View`，如下图所示。

<figure>
  <img src="../figs/check_floorplan_view.png" width=100%>
  <figcaption>Check Floorplan View</figcaption>
</figure>

可以看到初始的版图如下所示。

<figure>
  <img src="../figs/init_floorplan_view.png" width=80%>
  <figcaption>Initial Floorplan View</figcaption>
</figure>

除去Die Box之外，左侧的粉色正方形为RTL代码中的模块层次，正方形的大小表示了模块的预估面积。

??? Tip "TU & EU"
    正方形左上角`TU=64.7%`为Target Utilization (TU)，是指所有的标准单元和Macro的面积除以版图的面积。
    此外，还有Effective Utilization (EU)，在整体版图面积的基础上除掉了Placement，Routing Blockage等其他阻碍物的面积，Innovus默认不会显示EU。

Die Box右侧为该数字模块中例化的IP核，在该案例中包括若干SRAM和2个CIM Macro。

!!! Tip "检查Macro"
    在这个时候可以检查此前例化的SRAM等IP是否有成功导入Innovus。
    有时因为文件路径设置错误，会出现没有成功例化的情况。

#### 给Macro设置别名

``` tcl
# Main Memory
set mainmem i_sram

# D$
set dcache_tag0 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_wt_dcache_i_wt_dcache_mem/gen_tag_srams[0].i_tag_sram
set dcache_tag1 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_wt_dcache_i_wt_dcache_mem/gen_tag_srams[1].i_tag_sram
set dcache_data0 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_wt_dcache_i_wt_dcache_mem/gen_data_banks[0].i_data_sram
set dcache_data1 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_wt_dcache_i_wt_dcache_mem/gen_data_banks[1].i_data_sram

# I$
set icache_tag0 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_cva6_icache/gen_sram[0].i_tag_sram
set icache_tag1 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_cva6_icache/gen_sram[1].i_tag_sram
set icache_data0 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_cva6_icache/gen_sram[0].i_data_sram
set icache_data1 i_ariane/i_cva6/gen_cache_wt.i_cache_subsystem/i_cva6_icache/gen_sram[1].i_data_sram

# CIM Macro
set dcim_macro0 i_ariane/gen_coprocessor.i_in_pipeline_coprocessor/u_CIM_block_wrapper/CIM_core_unit_inst_DCIM_macro_1_inst
set dcim_macro1 i_ariane/gen_coprocessor.i_in_pipeline_coprocessor/u_CIM_block_wrapper/CIM_core_unit_inst_DCIM_macro_2_inst
```

鼠标左键单击某一个Macro，并按`Q`查看该Macro的属性，可以看到该Macro的名称，通常由RTL模块名称和IP名称组合而成。
如上所示，Macro的名称通常比较长，为了后续脚本简洁，给Macro设置简短的别名。（例如`icache_data0`, `icache_data1`）

#### 摆放Macro

``` tcl
# place Main Memory
placeInstance [set mainmem] 20 400 R180

# place CIM Macro
set basex 500
set basey 500
set deltay 350
placeInstance [set dcim_macro0] $basex [expr $basey + $deltay * 0]
placeInstance [set dcim_macro1] $basex [expr $basey + $deltay * 1]

# place I$
set basex 500
set basey 350
set deltax 150
set deltay 250
placeInstance [set icache_tag0] [expr $basex + $deltax * 0] [expr $basey - $deltay * 0] R180
placeInstance [set icache_tag1] [expr $basex + $deltax * 1] [expr $basey - $deltay * 0]
placeInstance [set icache_data0] [expr $basex + $deltax * 0] [expr $basey - $deltay * 1] R180
placeInstance [set icache_data1] [expr $basex + $deltax * 1] [expr $basey - $deltay * 1]

# place D$
# not shown for simplicity
```

命令格式为：`placeInstance instance_name <location> <orientation>`

* `instance_name`：想要摆放的Macro名称。可以使用别名，例如：`[set icache_data0]`或`$icache_data0`；
* `<location>`：有X和Y两个数值，分别表示Macro左下角的宽度方向和高度方向的坐标；
* `<orientation>`：设置Macro的摆放方向，可以选择`R0`, `R90`, `R180`, `R270`, `MX`, `MX90`, `MY`, `MY90`。

!!! Warning "Macro摆放方向"
    由于22nm工艺中栅极必须纵向摆放，所以在摆放Macro时只能选择`R0`, `R180`,`MX`, `MY`。

摆放好所有Macro之后的版图如下所示。

<figure>
  <img src="../figs/place_macro.png" width=80%>
  <figcaption>Layout after placing macros</figcaption>
</figure>


### 执行`add_halo_routeblk.tcl`

### 执行`global_net_connect.tcl`

### 执行`add_pin.tcl`

### 执行`add_endcap_wellcap.tcl`

### 执行`add_power_ring.tcl`

### 执行`add_power_stripe.tcl`

### 执行`place.tcl`

### 执行`cts.tcl`

### 执行`route.tcl`

### 修DRC报错

### 执行`add_core_filler.tcl`

### 执行`add_PG_pin.tcl`

### 执行`gen_files.tcl`