# 4. 数字子系统的物理设计（重构版）

!!! Warning
    Under development!

!!! tip "TLDR"
    1. 模板文件路径：`/work/home/limingxuan/common/SOC_CVA6/`

## 4.1 模板文件

## 4.2 物理设计流程

物理实现（后端）主要分为如下几个步骤：

- Floorplan
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

### Floorplan

布局规划（Floorplan）决定了**数字逻辑模块（soft module）**的位置、尺寸和形状以及**硬宏（hard macro）**的放置。
布局规划包括确定整体尺寸、I/O 引脚布局、凸块（bump）分配（倒装芯片，flip chip）、电源规划等。
布局规划与布局（placement）、早期全局布线（early global routing）相结合，是一个**迭代设计**过程。

*查看 Floorplan*

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

Floorplan 左侧所有 RTL 模块都拥有各自的 Floorplan 约束，可以通过在 GUI 上右键点击模块选择 `Attrbite Editor -> Constraint Type` 查看或者修改。
约束一共有如下几种：

- `None`：该模块未被预放置（pre-placed）。模块的放置不受任何限制。
- `Guide`：该模块被预放置，用以引导该模块中的标准单元的摆放。
- `Fence`：该模块的标准单元必须摆放在指定的位置。
- `Region`：同 Fence 约束，同时其他模块的标准单元也可以摆放在该区域内。
- `Soft Guide/Cluster`：约束相比 Guide 更加宽松。

和左侧的 RTL 模块类似，右侧的 IP 硬核也有各自的 Floorplan 约束，可以通过右键点击 Macro 选择 `Attrbite Editor -> Constraint Type` 查看或者修改。
约束一共有如下几种：

- `unplaced`：该硬核未被预放置。
- `fixed`：该硬核被预放置，不能通过自动化工具移动，但是可以通过交互指令移动。
- `placed`：该硬核被预放置，但是可以通过自动化工具或者交互指令移动。
- `cover`：该硬核被预放置，但是不能移动。
- `softFixed`：该硬核被预放置，只能在合法化的过程中被移动。

*设置 Floorplan*

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
