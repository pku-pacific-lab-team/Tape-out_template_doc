# 3. 数字子系统的门级仿真

逻辑综合后会生成门级网表，我们需要对其进行仿真以验证综合后的网表功能正确，仿真的具体流程和综合前的[行为级仿真](1_behavioral_simulation.md)类似。

## 3.1 模板文件

我们使用开源 CPU CVA6 以及 AXI 总线构成的 SoC 作为门级仿真模板示例，其文件夹路径为：

```
/work/home/limingxuan/common/SOC_CVA6/
```

## 3.2 运行仿真

在 `SOC_CVA6` 主目录下运行如下指令即可完成门级仿真。

```
b make gate_verdi TOP=<top_module_name>_tb
```

如果你不想使用 Verdi 查看波形文件，可以使用如下指令。

```
b make gate_vcs TOP=<top_module_name>_tb
```

运行仿真时，脚本会根据 `utils/user_define.tcl` 中的配置以及网表文件 `syn/<top_module_name>/<top_module_name>_postsyn.v` 自动生成源文件列表 `sim/build/gate_filelist.f`，然后进行编译和仿真。

!!! warning "网表文件路径"
    脚本只会搜索网表文件的默认位置，如果你改变了网表文件的位置，请手动修改 `sim/Makefile` 文件。

!!! bug "sdf 反标的时序仿真"
    目前脚本不支持 sdf 反标的时序仿真，大概率会出现**大量不定态**！

## 3.3 查看波形

在 Verdi 中，点击波形区域，使用快捷键 `r` 可以恢复波形，加载 `sim/gate_soc_signal.rc` 文件可以查看 CPU 指令波形。

考虑到实际流片时 SRAM 主存上电后是随机数，因此在 bootrom 中会将主存基地址处的 256-bit 数置 0。
因此加载 `sim/gate_soc_signal.rc` 文件后，可以看到 CPU 在执行主存基地址处的指令后会触发异常，进而在终端异常处理程序处循环。