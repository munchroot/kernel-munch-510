# K40S munch — 5.10+ 学习基座（阶段 2，v1）

目标：Redmi K40S（munch / sm8250）从零跑通 5.10+ 云编译，理解原理、掌握方法。
当前系统：澎湃 OS 4 / Android 17，测试一律先 `fastboot boot`，不覆盖原分区。

## 基座（已查实，2026-11-28）

- 内核树：`https://gitlab.nura.eco/soc/qualcomm-sm8250/linux.git`，分支 `7.2.0`
- HEAD：`71f4068cb254730bcf334cde9bdd6ca183e4fcd0`（2026-08-29），Makefile 7.2.0
- 配置：树内无 sm8250 defconfig，用 pmaports 的
  `device/community/linux-postmarketos-qcom-sm8250/config-postmarketos-qcom-sm8250.aarch64`
  拷为 `.config` 后 `olddefconfig`。raw URL 在调研时未直接验证通过，
  工作流里先 curl，失败则浅克隆 pmaports 再拷，都失败就 loud fail。
- 工具链：Ubuntu 24.04 自带 Clang 18（内核 7.2 最低要求 Clang 17.0.1）。
  alioth 的 Nix 钉死 LLVM 22，那是 alioth 的选择，不是内核要求。

## v1 的诚实范围

- 纯主线 7.2.0 + postmarketOS sm8250 配置，**不含 KernelSU / SUSFS / BakaSU**。
- 产物：`Image` / `Image.gz`、全部通用 sm8250 `.dtb`、`.ko` 模块包、`config-built.aarch64`。
- **没有 munch dtb**：树里没有 munch DTS，v1 只证明“编得出”。
  munch dtb 要等 `dts/munch/sm8250-xiaomi-munch.dts` 写好放进本仓库，
  工作流里的 “munch DTS hook” 那一步会自动把它编进去。
- 没有 `dtbo.img`（主线只产单个 `.dtb`），没有 AnyKernel3 zip，
  `fastboot boot` 用的 boot.img 组装（mkbootimg）是阶段 3 的事。

## 目录

- `.github/workflows/build-5.10-munch.yml` — 唯一的工作流，手动触发（workflow_dispatch）
- `patches/` — 以后放 munch 补丁（`*.patch`，工作流会自动 `git apply`），现在是空的
- `dts/munch/` — 以后放 `sm8250-xiaomi-munch.dts`，现在是空的

## 写 munch DTS 前必须先补齐的三项（未查实，别猜）

1. munch 的调试串口号与原机 console 参数（alioth 用 uart12，但 munch 未核实）
2. munch 的 reserved-memory 地址表（alioth 的地址绝不能照抄，填错起不来）
3. munch 触控的总线 / 芯片型号 / IRQ 与复位 GPIO（alioth 是 SPI 的 FT8756，munch 是 FocalTech，未核实是 SPI 还是 I2C）

## 下一步（等用户确认）

1. 确认推到 GitHub 哪个仓库（新建一个，还是用现有仓库）。
2. 推上去后手动触发首跑，看真实报错补齐 apt 包名 / config URL 等未查实项。
3. 首跑编过后，再写第一版最小 munch DTS（只求编过 + 串口/pstore，不求亮屏）。
