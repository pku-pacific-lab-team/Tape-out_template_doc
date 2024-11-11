# Innovus 入门

物理实现（后端）主要分为如下几个步骤：

- 
- Place
- Clock Tree Synthesis (CTS)
- Route
- Metal Fill


<figure>
  <img src="../figs/timing_closure_flow.png" width=80%>
  <figcaption>Recommended Timing Closure Flow</figcaption>
</figure>

## 数据准备

后端流程需要的数据如下：

- Timing Libaraies：`.lib` 文件，所有单元的时序信息。
- Physical Libraries：`.lef` 文件，所有单元的物理信息。
- Verilog Netlist：`.v` 文件，逻辑综合后的网表。
- Timing Constraints：`.sdc` 文件，时序约束。
- Multi-Mode Multi-Corner (MMMC) Setup for Timing：`.tcl` 文件，多模式多角约束。

Innovus 拥有全局 TCL 变量，这些变量都形如 `init_*`，可以通过 `set` 命令修改。
`saveDesign` 命令会将这些全局变量保存到 `.globals` 文件中。





## backup

foundation flow

hierachical flow


已修改的文件夹：

- `utils`
- `pnr`
- Makefile

