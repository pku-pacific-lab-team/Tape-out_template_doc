# 11. RISC-V CPU 调试

为了便于 CPU 的调试，ISA 会规定 Debug Module 的设计规范，以便于调试器（Debugger）与 CPU 之间的通信。
RISC-V 的 Debug Module 也是如此，详情可以查阅[官方设计说明文档](https://riscv.org/wp-content/uploads/2019/03/riscv-debug-release.pdf)。

<figure>
  <img src="../figs/riscv_debug.png" width=70%>
  <figcaption>RISC-V Debug System Overview</figcaption>
</figure>

与我们直接交互的软件为**调试器（Debugger）**（例如 GDB），它运行在调试主机上。
调试器与**调试转换器（Dubug Translator）**（例如 OpenOCD）通信，调试转换器与**调试传输硬件（Debug Transport Hardware）**（例如 USB-JTAG 适配器）通信。
调试传输硬件通过 JTAG 信号连接到测试平台（待测试的SoC）的调试传输模块（DTM）。DTM 使用调试模块接口（DMI）提供对调试模块（DM）的访问。

我们调试 RISC-V CPU 所需的工具如下：

- **Debugger**：RISC-V GDB
- **Debug Translator**：OpenOCD
- **Debug Transport Hardware**：USB-JTAG 适配器

## 11.1 安装 RISC-V Toolchain

## 11.2 安装 OpenOCD

## 11.3 调试 RISC-V CPU
