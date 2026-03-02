# 1. 数字子系统的行为级仿真

完成 RTL 设计后需要进行**行为级仿真**，以验证设计的功能是否符合预期。
在将设计的模块集成到 SoC 之后，也需要对整个 SoC 进行仿真，通过 **CPU 指令**的方式验证整个 SoC 的功能。
我们使用 synopsys 的 **VCS** 工具进行仿真、**Verdi** 工具查看波形文件。

!!! tip "TLDR"
    1. 仿真脚本：`./Makefile`、`sim/Makefile`
    2. 仿真命令：`make verdi`

## 1.1 模板文件

我们使用开源 CPU CVA6 以及 AXI 总线构成的 SoC 作为模板示例，其文件夹路径为：

```
/project/common/block_flow_[12|22]nm_v1/
```

该文件夹的结构为：

```
$ROOT
├── src
│   ├── filelist.f                  # Filelist for RTL
│   └── ...
├── sim
│   ├── soc_tb.sv                   # SoC Testbench
│   ├── init_mem.hex                # Memory Initialization File
│   └── Makefile                    # Simulation Script
├── Makefile                        # Top-level Makefile
├── ...
...                                 # Other Folders/Files
```

!!! question "集成到 SoC"
    参考[系统集成](../frontend/rtl_design/system_integration.md)章节，将你的模块集成到 SoC 中，进行 SoC 级别的行为级仿真。

## 1.2 添加源文件

在 `src/filelist.f` 中添加你的子模块源文件。

如果在子模块中例化了 SRAM IP，或者其他的定制 IP，则不需要在 `src/filelist.f` 中添加行为级模型源文件，但是需要在 `sim/Makefile` 中通过 `SRC_LIST += <your_ip_behavior_model>.v` 的形式添加。

## 1.3 编写 testbench

在 `sim/soc_tb.sv` 中编写 testbench。
对于系统仿真来说，testbench 只需要提供时钟、复位信号以及**仿真时长**。
如果你只想仿真子模块，在 testbench 中实例化需要仿真的子模块即可。

## 1.4 运行仿真

在 `$ROOT` 主目录下运行如下指令即可完成 RTL 编译以及查看生成波形文件。

```csh
make verdi TOP=<top_module_name>_tb
```

如果你不想使用 Verdi 查看波形文件，可以使用如下指令。

```csh
make vcs TOP=<top_module_name>_tb
```

运行仿真时，脚本会创建 `sim/build/` 文件夹，所有运行仿真生成的文件都会放在这个文件夹中。

!!! danger "build 文件夹"

    请不要在 `build` 文件夹中保存任何文件！
    每一次逻辑综合都会**清空**该文件夹。

对于子模块通常会进行多次行为级仿真，可以使用快捷键 `Shift + s` **保存需要查看的波形**，以便后续仿真直接一键加载。
在 Verdi 中，点击波形区域，使用快捷键 `r` 可以**恢复波形**，加载 `sim/soc_signal.rc` 文件可以查看 CPU 指令波形。

!!! question "Makefile 内容"
    非常建议你**阅读** Makefile 脚本，以便更好地理解仿真的过程。


!!! Bug "FIXME!!!"
    更详细地介绍 vcs、verdi 脚本以及如何使用。
