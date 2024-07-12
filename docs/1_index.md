# Tape-out模板说明文档

因为流片流程较为繁杂且涉及到多个工具以及相关文件，本文整理了一份数字芯片后端设计流程的文件模板，并对相关命令和文件进行了简要说明和解释。

我们希望通过模板文件和说明文档，帮助后续接触该模板的朋友们快速熟悉数字后端设计流程。然而，这份说明文档不可能做到面面俱到，要更加深入地了解数字后端设计流程，还需要阅读EDA工具的用户手册和其他官方文档，并通过实践和不断试错来积累经验。

![数字芯片设计流程](./figs/digital_design_flow.png)


> 注意：该文档还在开发之中，若出现错误或纰漏，还请在GitHub仓库中提出Issue！

## 模板文件结构

总共有**两份不同**的流片模板文件，一份用于**数字子系统**的后端设计，一份用于**顶层模块**的后端设计。
因为学术流片大多是[MPW](https://en.wikipedia.org/wiki/Multi-project_wafer_service)（多项目晶圆服务），每个流片项目最起码是一个数字子系统，甚至可能由多个层级的数字子系统嵌套而成，而多个流片项目共用一个顶层模块，从而给顶层模块的设计与规划带来了新的挑战。

* **数字子系统**的流片模板位于：`/work/home/ztzhu/tapeout_template/submodule/`
* **顶层模块**的流片模板位于：`/work/home/ztzhu/tapeout_template/top/`

在该文档后续，将会先介绍**数字子系统**的流片模板，这是任何流片项目后端设计的基础，再介绍**顶层模块**的流片模板。

## EDA工具版本与路径说明

该流片模板依赖于以下的EDA工具：

### Cadence Genus

### Cadence Innovus 19.10

### ARM SRAM Compiler

通常使用ARM提供的`High Density Single Port SRAM SHVT MVT Compiler`。
二进制执行文件位于：` `

在命令行中输入该执行文件即可启动。

### ARM Register File Compiler

二进制执行文件位于：` `

在命令行中输入该执行文件即可启动。