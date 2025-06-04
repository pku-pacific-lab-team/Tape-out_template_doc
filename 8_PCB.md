# 8. PCB 绘制

板级电路在测试过程中为芯片提供电源、时钟、GPIO配置接口等。

## 整体流程

**原理图设计 —— PCB绘制 —— DRC检查 —— PCB下单 ——（SMT下单）**

原理图设计：设计电路板的原理图，这个过程中需要对调用的元器件进行选型。  
PCB绘制：电路板真实的物理实现，包括电路板的大小边界、元器件位置摆放、走线链接。  
DRC: 检查设计的PCB是否符合Design Rule、PCB是否与原理图一致。   
PCB下单：完成原理图PCB设计后，将设计好的PCB压缩可以通过嘉立创下单助手下单安排生产。 （空PCB无元器件）     
SMT下单：SMT是PCB厂家根据提供的PCB信息将所需要的元器件通过回流焊方式焊接。（PCB+元器件）

### 8.1 元器件与EDA工具

#### 元器件分类

为测试芯片绘制的PCB所用到的元器件通常可以分为三类：有源器件、无源器件、连接器。

有源器件：芯片、二极管、LED等需要供电的器件。  
无源器件：R、C、开关等无需供电的器件。  
连接器：排针，电源接口等传递电源和信号的接口。

!!! tip "提示"
    对于数字芯片测试来说，如果使用外部稳压源来供电的话，通常无需使用板级LDO/BUCK来产生电源，因此只需要用到无源器件和连接器。

#### 元器件封装

封装从结构上可以分为插件与贴片两种，插件是通过通孔焊接在PCB上，贴片则是通过PCB的焊盘进行焊接。插件的机械稳定性更好，贴片的寄生参数更小。

- 有源器件的封装种类繁多，不一一赘述，可以在立创商城自行选型。

- 无源器件封装  
RC通常使用贴片封装0402、0603、0805、1206。器件的尺寸与数字大小成正相关
<figure>
  <img src="../figs/mlcc_package.png" width=100%>
  <figcaption>RC Package</figcaption>
</figure>

- 连接器封装  
    - 2.54 - 1(2) - 2/3/4/8P 排针, 常用为GPIO配置引脚，低频低电流下也可以作为信号IO和电源输入。
        - 2.54：排针pin间距为2.54mm  
        - 1(2)：单排(双排)   
        - 2/3/4/8P：2/3/4/8个pin

    - 5557 - 2P 电源接口
    - SMA，可用作高频信号传输，例如时钟

#### EDA工具

常用的PCB设计EDA工具主要是Altimu Designer，嘉立创EDA。Altimu Designer在北大软件中心中没有提供，需要从其他途径下载，嘉立创EDA可以免费试用。下面教程主要是基于Altimu Designer，但两者的操作是很类似的。

### 8.2 元件库添加
在进行原理图与PCB绘制前，我们需要添加元件库（R，C，芯片， 接口等等）。一个元器件主要包括Schematic与Footprint两部分。元器件引脚在schematic中的标号与Footprint的焊盘是一一对应的。元器件可以自行绘制或者通过嘉立创下载进行添加。

#### 元件库导入
元件库分为.SCHLIB, .PCBLIB, .INTLIB，其中.SCHLIB只有元件的schematic，.PCBLIB只有元件的pcb，.INTLIB则是两者都有。

1. 在右下角的 **Panels** 中把 **Components**选中，选择 **File-based Libraries Preferences**

<figure>
  <img src="../figs/add_lib.jpg" width=100%>
  <figcaption>Open Lib</figcaption>
</figure>

2. 选择Lib所在的路径然后install

<figure>
  <img src="../figs/install_lib.jpg" width=100%>
  <figcaption>Install Lib</figcaption>
</figure>

#### 添加元器件至元件库
如果元件库中没有想要的元件，需要自行添加，常用的方法是通过立创商城添中找到它，点击数据手册，在网页版嘉立创中打开它的封装，导出他的Schematic与Footprint为Altimu Designer。

<figure>
  <img src="../figs/add_new_element.jpg" width=100%>
  <figcaption>Add new component</figcaption>
</figure>

对于导出的Schematic/Footprint，在Altimu designer中打开它后在 **Tools-Make Schematic/PCB Library**      

打开对应的Library，copy对应的component，然后在右侧的**Components**中选择对应的库，随便找一个元件右键选择**Edit**来打开这个库，再将刚刚复制的component粘贴进去即可。

<figure>
  <img src="../figs/copy_component.jpg" width=100%>
  <figcaption>Copy component</figcaption>
</figure>

<figure>
  <img src="../figs/paste_component.jpg" width=100%>
  <figcaption>Paste Component</figcaption>
</figure>

### 8.3 原理图绘制
**下面操作基于Altium Designer软件进行（嘉立创的操作也是类似的，可能快捷键有所差别）**

1. File-New-Project 创建一个project
<figure>
  <img src="../figs/creat_proj.jpg" width=100%>
  <figcaption>Creat project</figcaption>
</figure>

2. 在创建的project下添加新的schematic与PCB
<figure>
  <img src="../figs/add_pcb.jpg" width=100%>
  <figcaption>creat new schematic and PCB</figcaption>
</figure>

3. 现在可以在schematic绘制电路图，P键可以弹出绘制的选择面板，**Place Part**会在右侧现实component的SCHLIB，将想要的器件拖到schematic中，然后进行连接。

4. 连接可以通过**Wire**或者**Net Label**进行，**Power Port**用来识别电源端口作用与Label相同 
*选中器件后空格键可以对器件进行旋转*

<figure>
  <img src="../figs/place_part.jpg" width=100%>
  <figcaption>Place Part</figcaption>
</figure>

5. 对于元器件，右键可以查看其Properties，修改Comment（PCB的丝印）以及Footprint



### 8.4 PCB绘制

1. 在进行PCB设计前我们需要先修改一下Design rule来满足我们的需求，通过**Design-Rules**打开面板，主要是修改间距与连接方式，因为默认设置很保守且适用面也较窄。（后续设计过程中也可以进一步继续调整满足需求）
    - 修改**Electrical-Clearance**，调整clearance，PCB中的间距不太重要，可以尽可能往小设，不会出错。
    - 修改**Electrical-Short Circuits**，勾掉Allow Short Circuit。
    - 修改**Routing-Width/Routing Via Style**，调整minimum（6）与maximum（100）范围。
    - 修改**Plane-Power Plane Connect Style/Polygon Connect Style**，修改connect style为Direct Connect。
    - 修改**Manufacturing-Hole Size/Silk To Solder Mask Clearance/Silk To Silk Clearance**，修改clearance，其中silk的clearance无所谓，即使重合也没事。

<figure>
  <img src="../figs/pcb_rules.jpg" width=100%>
  <figcaption>PCB Rules</figcaption>
</figure>

2. 设置PCB层数：PCB我们可能用到2层板或者4层板，2层板组合为signal-signal，4层板组合常用signal-power-ground-signal。其中signal layer是正铜层所绘制的为真正的线路，plane layer是负铜层所绘制的是没有线路的地方。
在**Design-Layer Stack Manager**中可以添加层。推荐使用4层板。

<figure>
  <img src="../figs/pcb_layer.jpg" width=100%>
  <figcaption>PCB Layer</figcaption>
</figure>

3. 定义PCB形状：选中KeepOut Layer或者Mechanical Layer，用wire画出想要的形状后，选中所有，**Design-Board Shape-Define Board Shape from Selected Objects** 

!!! tip "提示"
    COB对PCB尺寸有要求，10 cm x 15 cm


4. 导入Schematic的器件至PCB **Design-Import Changes from xxx**，同时也可以反向将PCB中的修改同步至Schematic **Design-Update Schematics in xxx**。

5. 现在可以进行PCB绘制了，主要就是器件摆放，连线，以及给电源平面铺铜。下面是一些常用的快捷键：
    - 1 2 3切换视图
    - q 切换公制/英制
    - ctrl + F 翻面
    - ctrl + M 尺子
    - shift + s 切换多层还是单层显示, ctrl + shift + 鼠标滑轮切换层/键盘"-", "+"
    - ctrl + w 走线
    - shift + w 切换线宽
    - P + R 铺铜
    - shift + 空格 切换走线模式（45°/45°圆角/90°）

6. 铺铜设置，调整为**Pour Over All Same Net Objects**并且勾选去除死铜。然后对铺铜进行平滑倒角，按住角 拖动（shift + 空格调整形状）。
完成设计后最后在Top与Bottom layer空余地方上都铺上GND，即拉一个很大的铺铜罩住整个板子。
（**右键-Polygon Actions-Polygon Manager**可以设置不同网络铺铜的优先级）

<figure>
  <img src="../figs/smooth.jpg" width=100%>
  <figcaption>Polygon Smooth</figcaption>
</figure>

7. 缝合孔：在TOP/BOTTOM 的signal layer以及Power/Ground plane layer中都会存在power/gnd的铺铜，同一网络在不同layer的铺铜层需要用缝合孔进行连接。   
**Tools-Via Stitching/Shielding-Add Stitching to Net...**