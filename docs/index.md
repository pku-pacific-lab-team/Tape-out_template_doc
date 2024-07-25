# Tape-out模板说明文档

因为流片流程较为繁杂，涉及到多个EDA工具以及相应的脚本文件，本文档中整理了一份**适用于TSMC 22nm**的数字芯片后端设计流程文件模板，并对相关命令和脚本文件进行了简要说明和解释。

我们希望通过模板文件和说明文档，帮助后续接触该模板的朋友们快速熟悉数字后端设计流程。然而，一份说明文档不可能做到面面俱到，要更加深入地了解数字后端设计流程，还需要阅读EDA工具的用户手册和其他官方文档，并通过实践和不断试错来积累经验。

<figure>
  <img src="./figs/digital_design_flow.png" alt="数字芯片设计流程">
  <figcaption>数字芯片设计流程</figcaption>
</figure>

> 注意：该文档还在开发之中，若出现错误或纰漏，还请在GitHub仓库中提出Issue！

## 模板文件结构

因为学术流片大多是[MPW](https://en.wikipedia.org/wiki/Multi-project_wafer_service)（多项目晶圆服务），每个流片项目最起码是一个数字子系统，甚至可能由多个层级的数字子系统嵌套而成，而多个流片项目共用一个顶层模块，从而给顶层模块的设计与规划带来了新的挑战。

??? info "扩展：全掩膜"
    与 MPW 相对的流片模式为 Full-Mask（全掩膜），即一整块晶圆只有一个项目，通常用于大规模生产。

因此，在此整理归纳出**两份不同**的流片模板文件，一份用于**数字子系统**的后端设计，一份用于**顶层模块**的后端设计。

* **数字子系统**的流片模板位于：`/work/home/ztzhu/tapeout_template/submodule_tapeout/`
* **顶层模块**的流片模板位于：`/work/home/ztzhu/tapeout_template/top_io_tapeout/`

在该文档后续章节中，将会先介绍**数字子系统**的流片模板，这是任何流片项目后端设计的基础，再介绍**顶层模块**的流片模板。

## EDA工具介绍

该流片模板主要依赖于以下的EDA工具：

### Cadence Genus 19.12

在该流片模板文件中，我们主要依赖于`Makefile`和`TCL`自动化脚本文件进行逻辑综合，一般情况下不需要单独启动Genus综合工具。如需进行较为细致的调试，在终端输入`b genus`启动Genus综合工具。
默认情况下在当前目录下生成`genus.log`和`genus.cmd`日志文件，分别记录了Genus的终端输出和用户输入的命令。

为了更加全面了解Genus综合工具，可以查看Cadence的官方说明手册和文件：

* **Genus Synthesis Flow Guide**: `/cadtools/cadence/genus19.12/doc/genus_start/genus_start.pdf`
* **Genus User Guide**: `/cadtools/cadence/genus19.12/doc/genus_user/genus_user.pdf`
* **Genus Command Reference**: `/cadtools/cadence/genus19.12/doc/genus_comref/genus_comref.pdf`

### Cadence Innovus 20.10

该流片模板文件中，将主要围绕Innovus工具进行数字模块的后端设计。在终端中输入`b innovus`启动Innovus工具，默认情况会自动启动GUI界面，有助于我们观察后端版图的各类情况，方便迭代设计与优化。
使用以下命令可以打开、关闭GUI界面。
```tcl
# Turn on GUI
enc::gui_on 

# Turn off GUI
enc::gui_off 
```

大部分的设计流程使用Innovus的终端输入`TCL`脚本命令。
若使用GUI界面进行操作，相应的操作也会自动转化成`TCL`指令，可以打开`innovus.cmd`查看GUI界面的操作与Innovus指令的对应关系。

Cadence的官方说明手册和文件：

* **Innovus User Guide**: `/cadtools/cadence/innovus20.10/doc/innovusUG/innovusUG.pdf`
* **Innovus Error Messege**: `/cadtools/cadence/innovus20.10/doc/innovuserrmsg/innovuserrmsgTOC.html`

### ARM SRAM Compiler

许多数字模块中依赖于较大规模的寄存器堆/SRAM高速缓存，这些模块需要替换成专门的IP核，而不是使用RTL代码直接综合，从而可以显著减小模块面积。

替换SRAM时，我们通常使用ARM提供的`High Density Single Port SRAM SHVT MVT Compiler`。

**二进制执行文件**位于：

```
/work/home/tyjia/common/TSMC_22NM_ULL/CA001/arm/tsmc/cln22ul/sram_sp_hde_shvt_mvt/r6p0/bin/sram_sp_hde_shvt_mvt
```

在命令行中输入该执行文件即可启动。

**用户手册**位于：

```
/work/home/tyjia/common/TSMC_22NM_ULL/CA001/arm/tsmc/cln22ul/sram_sp_hde_shvt_mvt/r6p0/doc/sram_sp_hde_shvt_mvt_userguide.pdf
```

### ARM Register File Compiler

替换寄存器堆时，我们使用ARM提供的`High Density Single Port Register File SHVT MVT Compiler`。

**二进制执行文件**位于：

```
/work/home/limingxuan/common/TSMC_22NM_ULL/arm_rf_sp_shvt_mvt/tsmc/cln22ul/rf_sp_hde_shvt_mvt/r3p1/bin/rf_sp_hde_shvt_mvt
```

在命令行中输入该执行文件即可启动。

**用户手册**位于：

```
/work/home/limingxuan/common/TSMC_22NM_ULL/arm_rf_sp_shvt_mvt/tsmc/cln22ul/rf_sp_hde_shvt_mvt/r3p1/doc/rf_sp_hde_shvt_mvt_userguide.pdf
```

### Cadence Virtuoso 6.1.8

在进行数字模块的DRC与LVS检查时，会使用到Cadence Virtuoso工具。在终端中输入`b virtuoso`打开Virtuoso工具。

### Synopsys VCS

### Synopsys Verdi