# Tape-out模板说明文档

因为流片流程较为繁杂，涉及到多个 EDA 工具以及相应的脚本文件，本文档中整理了一份**适用于 TSMC 22nm** 的数字芯片后端设计流程文件模板，并对相关命令和脚本文件进行了简要说明和解释。

我们希望通过模板文件和说明文档，帮助后续接触该模板的朋友们快速熟悉数字后端设计流程。然而，一份说明文档不可能做到面面俱到，要更加深入地了解数字后端设计流程，还需要阅读 EDA 工具的用户手册和其他官方文档，并通过实践和不断试错来积累经验。

??? info "22nm vs. 28nm"
    28nm和22nm属于同一个工艺节点，这意味着二者的区别**仅限于尺寸**，其他的例如 DRC、LVS 规则**完全相同**。EDA 工具中，无论是22nm还是28nm工艺，其坐标或者其他尺寸均为28nm下的尺寸。22nm工艺只会在流片的时候进行尺寸缩放（x0.855）。


<figure>
  <img src="./figs/digital_design_flow.png" width=80%>
  <figcaption>数字芯片设计流程</figcaption>
</figure>

!!! question "关于文档"
    该文档还在持续开发更新之中，若出现错误或纰漏，还请在 GitHub 仓库中提出 Issue！
    如果有改进意见，欢迎提交 Pull Request！

## 模板文件结构

因为学术流片大多是 [MPW](https://en.wikipedia.org/wiki/Multi-project_wafer_service)（多项目晶圆服务），每个流片项目最起码是一个数字子系统，甚至可能由多个层级的数字子系统嵌套而成，而多个流片项目共用一个顶层模块，从而给顶层模块的设计与规划带来了新的挑战。

??? info "扩展：全掩膜"
    与 MPW 相对的流片模式为 Full-Mask（全掩膜），即一整块晶圆只有一个项目，通常用于大规模生产。

因此，在此整理归纳出**两份不同**的流片模板文件，一份用于**数字子系统**的后端设计，一份用于**顶层模块**的后端设计。

* **数字子系统**的流片模板位于：`/work/home/ztzhu/tapeout_template/submodule_tapeout/`
* **顶层模块**的流片模板位于：`/work/home/ztzhu/tapeout_template/top_io_tapeout/`

在该文档后续章节中，将会先介绍**数字子系统**的流片模板，这是任何流片项目后端设计的基础，再介绍**顶层模块**的流片模板。
最后介绍流片之后的**封装测试**。

## 目录

* [一、Linux环境下EDA工具介绍](./0_eda.md)
* 二、前端设计
    * [1. 数字子系统的行为级仿真](./1_submodule_behavioral_simulation.md)
    * [2. 数字子系统的逻辑综合](./2_submodule_synthesis.md)
    * [3. 数字子系统的门级仿真](./3_submodule_gate_level_simulation.md)
* 三、后端实现
    * [4. 数字子系统的后端设计](./4_submodule_implementation.md)
    * [5. 顶层模块的后端设计](./5_io.md)
    * [6. DRC/LVS验证](./6_lvs_drc.md)
    * [7. 拼版](./7_final_layout.md)
* 四、回片测试
    * [8. PCB绘制](./8_PCB.md)
    * [9. 封装](./9_package.md)
    * [10. 测试](./10_testing.md)
* 五、附录
    * [11. RISC-V CPU 调试](./11_cpu_debug.md)
    * [12. C 代码编译](./12_assembly.md)
    * [13. WSL](./13_wsl.md)
    * [14. FPGA 验证](./14_FPGA_verification.md)
    * [15. PDK设置](./15_PDK_setup.md)
    * [16. Digital-CIM 使用规范](./DCIM_spec.md)
* [六、如何贡献](contribute.md)

## 更新日志

### 当前版本

!!! Bug "当前版本尚未公测"
    开发中，尚未公测，敬请期待！

### 历史版本

## 致谢

感谢所有为该文档提供帮助的朋友们以及 ChatGPT！

{{ git_site_authors }}