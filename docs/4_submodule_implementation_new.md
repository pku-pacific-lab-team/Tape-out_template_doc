# 4. 数字子系统的物理设计（重构版）

!!! Warning
    Under development!

!!! tip "TLDR"
    1. 模板文件路径：`/work/home/limingxuan/common/SOC_CVA6/`

## 4.1 模板文件

## 4.2 物理设计流程

物理实现（后端）主要分为如下几个步骤：

- Floorplan
- Powerplan
- Place
- Clock Tree Synthesis (CTS)
- Route
- Metal Fill

<figure>
  <img src="../figs/timing_closure_flow.png" width=80%>
  <figcaption>Recommended Timing Closure Flow</figcaption>
</figure>

后端流程需要的数据如下：

- Timing Libaraies：`.lib` 文件，所有单元的时序信息。
- Physical Libraries：`.lef` 文件，所有单元的物理信息。
- Verilog Netlist：`.v` 文件，逻辑综合后的网表。
- Timing Constraints：`.sdc` 文件，时序约束。
- Multi-Mode Multi-Corner (MMMC) Setup for Timing：`.tcl` 文件，多模式多角约束。

Innovus 拥有全局 TCL 变量，这些变量都形如 `init_*`，可以通过 `set` 命令修改。
`saveDesign` 命令会将这些全局变量保存到 `.globals` 文件中。

!!! Tip "GUI 操作与脚本命令"
    所有的 GUI 操作都有对应的脚本命令，可以通过 GUI 操作后查看 `pnr/logs/<top_module_name>.cmd` 得到对应的脚本命令。

### 4.2.1 Floorplan

布局规划（Floorplan）决定了**数字逻辑模块（soft module）**的位置、尺寸和形状以及**硬宏（hard macro）**的放置。
布局规划包括确定整体尺寸、I/O 引脚布局、凸块（bump）分配（倒装芯片，flip chip）、电源规划等。
布局规划与布局（placement）、早期全局布线（early global routing）相结合，是一个**迭代设计**过程。

#### 查看 Floorplan

在 Innovus GUI 界面右上角选择 `Floorplan View`，如下图所示。

<figure>
  <img src="../figs/check_floorplan_view.png" width=100%>
  <figcaption>Check Floorplan View</figcaption>
</figure>

可以看到初始的版图如下所示。

<figure>
  <img src="../figs/init_floorplan_view.png" width=80%>
  <figcaption>Initial Floorplan View</figcaption>
</figure>

中央为整个芯片的版图，分为 Core Box，IO Box 和 Die Box，如下图所示。

<figure>
  <img src="../figs/floorplan_box.png" width=70%>
  <figcaption>Floorplan Box Definition</figcaption>
</figure>

所有标准单元的宽度各不相同，但是高度均为 `$cell_height`（对于22nm工艺，即为0.7um，pre-shrink）。
在 Core box 的四周到 I/O 管脚之间通常留有一定的间距，用于摆放 Core Ring 和 I/O 管脚布线。

除去中央的 Die Box 之外，左侧的粉色正方形为 RTL 代码中的数字顶层模块，正方形的大小表示了模块的预估面积。

??? Tip "TU & EU"
    正方形左上角 `TU=64.7%` 为 Target Utilization (TU)，是指所有的标准单元和 Macro 的面积除以版图的面积。
    此外，还有 Effective Utilization (EU)，在整体版图面积的基础上除掉了 Placement，Routing Blockage 等其他阻碍物的面积，Innovus 默认不会显示EU。

Die Box 右侧为该数字模块中例化的 IP 硬核，在该案例中包括若干 SRAM 和2个 CIM Macro。

!!! Tip "检查 Macro"
    在这个时候可以检查此前例化的 SRAM 等 IP 是否有成功导入 Innovus。
    有时因为文件路径设置错误，会出现没有成功例化的情况。

Floorplan 左侧所有 RTL 模块都拥有各自的 Floorplan 约束，可以通过在 GUI 上右键点击模块选择 `Attrbite Editor -> Constraint Type`（快捷键 Q）查看或者修改。
约束一共有如下几种：

- `None`：该模块未被预放置（pre-placed）。模块的放置不受任何限制。
- `Guide`：该模块被预放置，用以引导该模块中的标准单元的摆放。
- `Fence`：该模块的标准单元必须摆放在指定的位置。
- `Region`：同 Fence 约束，同时其他模块的标准单元也可以摆放在该区域内。
- `Soft Guide/Cluster`：约束相比 Guide 更加宽松。

和左侧的 RTL 模块类似，右侧的 IP 硬核也有各自的 Floorplan 约束，可以通过右键点击 Macro 选择 `Attrbite Editor -> Constraint Type`（快捷键 Q）查看或者修改。
约束一共有如下几种：

- `unplaced`：该硬核未被预放置。
- `fixed`：该硬核被预放置，不能通过自动化工具移动，但是可以通过交互指令移动。
- `placed`：该硬核被预放置，但是可以通过自动化工具或者交互指令移动。
- `cover`：该硬核被预放置，但是不能移动。
- `softFixed`：该硬核被预放置，只能在合法化的过程中被移动。

#### 设置 Floorplan

一般情况下，我们使用如下命令设置 Floorplan。

```tcl
floorPlan -d {W H Left Bottom Right Top}
```

使用该命令设置版图大小总共需要6个参数，分别代表：

* 完整版图 (Die box) 的宽度。
* 完整版图的高度。
* Core box（用于摆放标准单元的版图部分）到I/O左侧边界的距离。
* Core box 到 I/O 底部边界的距离。
* Core box 到 I/O 右侧边界的距离。
* Core box 到 I/O 上方边界的距离。

一个初始的版图类似下图所示，设置该版图大小的命令为：`floorPlan -d 100 100 10 10 10 10`。

<figure>
  <img src="../figs/example_floorplan.png" width=80%>
  <figcaption>Example Floorplan</figcaption>
</figure>

??? Tip "根据标准单元密度自动设置版图大小"

    ```tcl
    floorPlan -su {aspectRatio [stdCellDensity [Left Bottom Right Top]]}
    ```

    - aspectRatio：版图的高宽比（高/宽）。
    - stdCellDensity：标准单元密度，即标准单元的个数。
    - Left Bottom Right Top：Core box 到 I/O 四个边界的距离。

    版图的面积会由如下公式计算得到：coreArea = stdCellArea ÷ stdCellDensity + macroArea。

#### 摆放 Macro

*脚本命令*

Macro 的名称可以在 `Floorplan View` 中右键点击 Macro 选择 `Attrbite Editor -> Name`（快捷键 Q）查看。
Macro 的名称通常比较长，为了后续脚本简洁，建议使用 TCL 命令 `set nickName realName` 给 Macro 设置简短的别名，例如 `set icache_data0 i_wt_dcache_i_wt_dcache_mem/gen_data_banks[0].i_data_sram`。

我们使用如下命令摆放 Macro。

```tcl
placeInstance instance_name location [orientation]
```

* `instance_name`：想要摆放的 Macro 名称。可以使用别名，例如：`$icache_data0`。
* `location`：有 X 和 Y 两个数值，分别表示 Macro 左下角的宽度方向和高度方向的坐标。
* `[orientation]`：（可选）设置 Macro 的摆放方向，可以选择 `R0`, `R90`, `R180`, `R270`, `MX`, `MX90`, `MY`, `MY90`，默认为 `R0`。

!!! Warning "Macro 摆放方向"
    由于 22nm 工艺中栅极必须纵向摆放，所以在摆放 Macro 时只能选择 `R0`, `R180`,`MX`, `MY`。

*GUI 操作*

点击工具栏中的 `Move/Resize/Reshape` 图标（快捷键 Shift+R），然后单击需要移动的 Macro（使用 Shift 多选），拖动到指定位置，再单击一次即可摆放。
该图标的次级菜单可以选择是否只能水平垂直移动。

<figure>
  <img src="../figs/macro_move.png" width=100%>
  <figcaption>Move/Resize/Reshape Widget</figcaption>
</figure>

<figure>
  <img src="../figs/move_orthogonal.png" width=100%>
  <figcaption>Move Restriction</figcaption>
</figure>

在摆放多个 Macro 时，可以使用工具栏中的 `Floorplan Toolbox`。
按照下图中的步骤 1，可以看到在中央视图的左上角会出现一个工具箱，选中需要操作的多个 Macro，然后点击该工具箱即可对选中的 Macro 进行对齐、等间距排列等操作。

<figure>
  <img src="../figs/floorplan_toolbox.png" width=100%>
  <figcaption>Floorplan Toolbox</figcaption>
</figure>

摆放完 Macro 之后，可以使用如下命令修改所有 Macro 的摆放属性。

```tcl
setInstancePlacementStatus -allHardMacros -status {fixed | placed | cover | softFixed}
```

摆放好所有 Macro 之后的版图如下所示。

<figure>
  <img src="../figs/place_macro.png" width=80%>
  <figcaption>Layout after placing macros</figcaption>
</figure>

#### 添加 Pin

对于大规模的数字模块，信号 I/O 管脚数量可能是成百上千的，手动编写命令设置每个管脚的摆放过于繁琐。
因此，我们使用 Innovus GUI 界面添加数字子系统的信号 I/O 管脚。

* 在上方工具栏选择 `Edit -> Pin Editor` 添加相应的管脚（详见下方 Pin Editor GUI 界面）。
* 打开 `pnr/logs/<top_module_name>.cmd` （Innovus 生成的日志文件）查看我们在 GUI 界面每一步操作所对应的命令，其中就有我们所需要的 `editPin` 命令。

<figure>
  <img src="../figs/pin_editor.png" width=80%>
  <figcaption>Innovus Pin Editor</figcaption>
</figure>

可以设想，如果管脚数量增加，例如有2组 64-bit 输入信号和1组 64-bit 输出信号，手写脚本命令过于繁琐，这也是我们在此使用 GUI 界面的原因。

!!! tip "关于 I/O 管脚金属层数的选择"
    在常规的数字芯片中，奇数层的为横向金属，偶数层为纵向金属（常称为**奇横偶纵**），因此对于 Top/Bottom 可以选择 M4/M6 等金属，Left/Right 选择 M3/M5 等金属。在上面 2-bit 加法器的例子中，每一边（Left/Top/Bottom）仅仅用到了一层金属，在管脚较多的情况下，可以将不同的管脚分配到同一条边的不同金属层。

!!! tip "关于数字子系统的 Power/Ground 的 I/O 管脚"
    数字子系统的 Power/Ground 管脚和不同信号线的管脚有所区别，往往是以顶层1-2层的电源网格的形式给数字子系统进行供电，因此在布局布线完成之后使用 `createPGPin` 命令生成，在[后续步骤](./4_submodule_implementation_deprecated.md#416-执行-add_pg_pintcl)做进一步介绍。

添加完 pin 后的版图如下图所示，每一个黄色的三角形代表一个 I/O 管脚，Zoom In 可以进一步看到每个管脚的名称，所在的金属层，以及管脚的具体形状 (Pin Width, Pin Depth)。

<figure>
  <img src="../figs/add_pin.png" width=80%>
  <figcaption>Layout after adding I/O pins</figcaption>
</figure>



#### 添加 Halo 和 Route Blockage

Halo 用于防止 Macro 四周摆放标准单元，Route Blockage 用于防止布线进入指定区域，两者都可以减缓布线阻塞。

#### 保存/恢复 Floorplan

*脚本命令*

```tcl
saveFPlan <design_name>.fp
loadFPlan <design_name>.fp
```

*GUI 操作*

<figure>
  <img src="../figs/save_floorplan.png" width=80%>
  <figcaption>Save and Load Floorplan</figcaption>
</figure>
