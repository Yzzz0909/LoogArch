# LoogArch

## 项目概述

LoogArch 是一个开源的国产 ISA 软核项目，灵感来源于 Loongson / LoongArch 架构，目标实现五级流水、软核 CPU、AXI 外设互联、FPGA 综合和 SDK 软件运行。从 RTL 到软件，再到仿真测试，覆盖了完整 SoC 开发流程。

## 代码结构

- `rtl/`
  - `ip/open-la500/`: 核心处理器设计（指令缓存、数据缓存、TLB、BTB、CSR、流水线五级、ALU/乘除法等）
  - `ip/APB_UART/`: UART 外设与 APB/AXI 桥接逻辑
  - `ip/Bus_interconnects/`: AXI 总线互联与 SRAM 适配器
  - `ip/ram_wrap/`: FPGA SRAM/缓存模型封装
  - `ip/confreg/`: 配置寄存器、数码管、按键去抖等板级控制模块
  - `ip/PLL_2019_2/`: 时钟生成模块

- `fpga/`
  - `create_project.tcl`, `constraints/`: FPGA 综合与约束脚本

- `sdk/`
  - `software/bsp/`: 运行时支持库与中断/定时函数
  - `software/examples/`: `hello_world`, `coremark`, `dhrystone`, 其他测试程序
  - `toolchains/`: 交叉编译链安装与配置说明

- `sim/`
  - `mycpu_tb.v`: 处理器测试平台
  - `sram.v`: SRAM 参考模型

- `doc/`
  - `集创赛龙芯中科杯初赛发布包说明.pdf`: 比赛题目与需求说明（核心需求来源）
  - `rtl/ip/open-la500/doc/`: Architectual文档，包含 `设计概述.md`, `前言.md`, `分支预测.md`, 以及架构图

## 任务完成情况（基于 doc 指导）

- 完成 CPU 五级流水实现：`if_stage.v`, `id_stage.v`, `exe_stage.v`, `mem_stage.v`, `wb_stage.v`。
- 实现指令缓存与数据缓存：`icache.v`, `dcache.v`，含缺失处理与AXI设备交互逻辑。
- 实现虚拟地址转换与异常机制：`tlb_entry.v`, `addr_trans.v`, `csr.v`, `mycpu_top.v`。
- 分支预测与流水恢复：`btb.v`, `分支预测.md`描述策略，已集成预测/更新/回退路径。
- ALU 与高性能运算单元：`alu.v`, `mul.v`, `div.v`，支持基本算术逻辑和多周期乘除。
- 外设与总线：`axi_bridge.v`, `axi2sram_*`，`AXI3`主控/从控协议连接缓存与 SOC 总线。
- UART 片上外设：`APB_UART` 下完整 UART 控制器 + FIFO + 传输协议。
- FPGA 适配：`fpga/create_project.tcl`, `constraints/soc.xdc` 实现 Vivado 约束与综合导出。
- 软件测试：`sdk/software/examples` 包含常见基准程序与上电功能验证代码。
- 仿真平台：`sim/mycpu_tb.v` + `sim/sram.v` 支持 RTL 功能验证。

## 关键模块概述（摘自 `rtl/ip/open-la500/doc`）

- `pfs/fs/ds/es/ms/ws` 五级流水级：负责指令流水的 `fetch/decode/execute/memory/writeback`。
- `btb`: 32 组 CAM + 8 深 RAS 返回地址栈，实现动态分支预测。
- `icache`: 2 路组相联，随机替换策略，`inst_addr_ok` 控制忙等。
- `dcache`: 读写混合，脏数据写回逻辑，dcache miss 通过 `axi_bridge` 调用外部内存。
- `tlb`: 32 项 CAM，指令/数据共享，支持缺页异常处理。
- 前递/阻塞：`es_to_ds_forward_bus`, `ms_to_ds_forward_bus`, `stage_allowin`, `stage_ready_go` 等信号
  实现流水冲突、冒险与停顿管理。

## 运行步骤

1. 安装交叉编译链
   - 修改 `sdk/toolchains/init.sh` 中 `CICIEC_WINDOWS_HOME` 路径
   - `cd sdk/toolchains && ./init.sh`
   - `loongarch32r-linux-gnusf-gcc --version`
2. 编译例程
   - `cd sdk/software/examples/hello_world && make`
3. 仿真验证
   - 使用 `iverilog/vvp` 或其他工具组合运行 `sim/mycpu_tb.v`，观察波形并调试
4. FPGA 综合
   - 在 Vivado 中运行 `fpga/create_project.tcl`
