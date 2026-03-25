# Copilot Workspace Instructions for LoogArch

## Purpose

This file configures in-repo agent behavior for the LoogArch CPU/SoC project.

- Project: Loongson-inspired RISC-V-ish/LoongArch open-source core and platform
- Scope: RTL (`rtl/`), FPGA integration (`fpga/`), SDK software (`sdk/software/`), and simulation (`sim/`)
- Goal: provide quick actionability for code changes, tests, and docs updates.

## 1. Discover Existing Conventions (already done)

- no `.github/copilot-instructions.md`, `AGENT.md`, or `AGENTS.md` existed before this file.
- primary README is minimal, most developer documentation exists in subfolders like `sdk/toolchains/README.md` and `rtl/ip/open-la500/README.md`.

## 2. Project Structure Overview

- `rtl/`: Verilog sources for CPU and peripherals
  - `ip/open-la500/`: CPU microarchitecture implementation (id/if/exe/mem/wb + cache/tlb/csr)
  - `ip/APB_UART/`: UART APB bridge and async FIFOs
  - `ip/Bus_interconnects/`: AXI interconnect and SRAM adapters
- `fpga/`: vendor-specific FPGA wrappers & constraints
- `sdk/`: embedded software examples and BSP
  - `sdk/software/examples/`: benchmarks and demos (coremark, dhrystone, hello_world)
  - `sdk/toolchains/`: setup scripts for cross-toolchain
- `sim/`: testbench and SRAM models

## 3. Build/Test Commands

### RTL simulation
- Likely with `iverilog`/`vvp` or vendor tool; no top-level script in repo.
- Suggested starting point: run `sim/mycpu_tb.v` and `sim/sram.v` with your simulator.

### FPGA flow
- Use `fpga/create_project.tcl` + `fpga/constraints/soc.xdc`.

### SDK (software)
1. Update `sdk/toolchains/init.sh` with `CICIEC_WINDOWS_HOME` path.
2. Run in Git Bash:
   - `cd sdk/toolchains && ./init.sh`
   - `loongarch32r-linux-gnusf-gcc --version`
3. Build examples:
   - `cd sdk/software/examples/hello_world && make`

## 4. Code-style / contribution guidelines

- Verilog modules use lower-case with underscores (e.g. `axi2sram_sp.v`).
- In CPU code, registers and constants are in `mycpu.h`, `csr.h`.
- Keep comments in English/Chinese matching existing style.

## 5. Typical tasks for Copilot agent

- Add/extend instructions in `rtl/ip/open-la500/` (fetch/decode/execute pipeline)
- Sync CSR, TLB, cache behavior in `csr.v`, `tlb_entry.v`, `dcache.v`.
- Implement peripheral access in `axi2apb.v`, `AxiCrossbar_1x4.v`.
- Data path bug fixes in `sim/mycpu_tb.v` + `sram.v`.
- Update docs in `doc/` and module `README.md` files.

## 6. Important notes

- There is no unified CI config; create one in `.github/workflows` if needed.
- `fpga/ip/open-la500/doc/` contains design discussions and should be referenced for semantics.

## 7. Example agent prompts

- "Add support for floating-point unit in `rtl/ip/open-la500` and provide a test case in `sim/mycpu_tb.v`."
- "Fix pipeline flush behavior in `mem_stage.v` when branch misprediction occurs."
- "Document how `axi2sram_sp.v` handshake signals map to AXI bursts."

## 8. Suggested next agent-customizations

- `/create-instruction cpu-pipeline`: focused on walkthroughs for `open-la500` pipeline stages.
- `/create-hook lint-verilog`: enforce naming/convention rules for `rtl/**` files.
- `/create-agent sim-workflow`: automate running simulation + checking waveform assertion failures.
