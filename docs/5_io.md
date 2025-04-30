# 5.顶层IO模块的后端流程

在进行模块层级的综合和后端之后，我们需要将不同的模块打包为一个完整的芯片，即进行芯片的**顶层设计**。芯片的顶层需要将之前已经完成的各个模块版图整合为一个完整的矩形，添加硬件意义上的输入输出端口（I/O Ring）和Seal Ring 以提供电气和物理的保护，并进行Dummy Fill以清除所有会影响到Tape-Out 的density DRC违例。

顶层设计相关的后端脚本路径位于/work/home/yysun/ARCHIVE/IO_Top/pnr/script/，其结构与模块后端脚本相同，在后端前的config修改等内容也相同，不再赘述。

顶层的后端流程大致可以分为Floorplan，布局布线，PAD Connection，SealRing添加与Dummy Fill几个步骤。

## Floorplan

在这个阶段，我们需要规划整个芯片所有输入输出信号和电源连接所在的位置，实例化并摆放对应的IO PAD

### 整体布局

在布局顶层PAD时，主要需要考虑如下因素：

1. 划分出的芯片单独供电区域是否都至少有一边达到了芯片的边缘IO Ring
2. 电源PAD的数量是否充足，能否相对均匀的覆盖所有的重要器件
3. 信号PIN的分布是否过于集中，导致某一侧Wire Bond 压力过大

!!! 在 Wire Bond时，每一个信号PIN需要占用封装管壳的一个输出管脚，同名的VDD信号可以公用一个管脚，而VSS则是 Down-Bond 到管壳底，不需占用管脚。由于封装管壳的管脚分布是均匀的（以QFN64为例，每边各有16管脚），如果信号PIN分布过于集中，可能导致一侧的管脚压力过大，使芯片Wire-Bond的位置偏离中心甚至不能正常在管壳中放置。

#### 电源域的划分

最终芯片测试时我们希望能够单独测量各个关键部分的能耗（如CIM Macro），因此需要单独对其供电，将其他器件的电流排除。实现单独供电有两种方式：

1. 将希望单独供电的电源接到模拟电源PAD上（PVDD2ANA）
2. 将IO Ring用PRCUT切断，使不同的数字电源域物理隔离

#### PAD的种类及数量要求

在物理划分出的每一个电源域上均需要至少有一组VDD_IO（PVDD2DGZ, PVSS2DGZ）以给IO供电，一个POC PAD（PVDD2POC）进行电压波动保护。除此之外，还需要若干给芯片内部Core区域供电的VDD，VSS PAD（PVDD1DGZ， PVSS1DGZ，PVDD2ANA等），考虑到IR drop等因素，Core电源PAD通常越多越好。

#### PAD的间距要求

我们所采用的Wire Bond PAD为 PAD52D6GU，这意味着两个相邻的信号/电源PAD的PITCH**不能低于52μm**，通常情况下我们选择60μm作为一个较为合适的间距。

### 信号PAD的实例化

示例的CVA6 CPU的顶层模块路径为
/work/home/yysun/ARCHIVE/IO_TOP/soc/src/soc_pad_wrapper.sv

其中定义了CPU 及一组DRAM Hyper Bus 所需的Signal IO PAD，可以根据实际需要进行增减。我们以JTAG端口的TCK为例说明端口含义

```verilog
PDDWUW0408SDGH_H i_jtag_tck_pad_i(  
//器件名称后的_H代表其放置在芯片的左右两侧，若需要在上下放置则需将_H改为_V
    .PU         (1'b0),         //PAD拉高，此处为无关信号
    .PD         (1'b0),         //PAD拉低，此处为无关信号
    .DS         (1'b0),         //输出驱动能力选择，输出时拉高
    .IE         (1'b1),         //输入使能，1有效
    .OEN        (1'b1),         //输出使能，0有效
    .I          (1'b0),         //片内需输出的信号，输入时接0
    .C          (jtag_tck_i),   //片内需输入的信号,输出时不接
    .PAD        (jtag_tck_pad_i)//Wire Bond端口
);
```

!!! 信号PAD的实例命名要求以**i_**起始,后接**PAD端信号名**以方便后端脚本进行批量布局

### 电源PAD的实例化以及PAD的摆放

根据脚本注释，对应修改/script/floorplan/floorplan.tcl中的相关指令完成电源PAD实例化以及批量PAD摆放

## 布局布线

## 版图

## SealRing

## Dummy

!!! Warning "Under development!"
