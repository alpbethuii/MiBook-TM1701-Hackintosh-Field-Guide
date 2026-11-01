# Xiaomi Mi Notebook Pro TM1701/TM1707 Hackintosh Field Guide

> **一台真实 TM1701 的长期工程记录：从 BIOS、OpenCore、ACPI、kext，到图形、音频、无线、USB、睡眠与 S4。**
>
> 目标不是再发一份“能开机的 EFI”，而是把一台机器从能启动，逐步收敛到可解释、可复现、日常可用，并且把每一次失败、回滚和最终取舍都留下证据。

[English overview](README_EN.md) · [版本选择历史](docs/12-os-version-history.md) · [完整工程时间线](docs/13-engineering-timeline.md) · [硬件适用范围](docs/01-hardware-and-compatibility.md) · [从零安装](docs/02-installation.md) · [S4 深度案例](docs/case-studies/s4-sleep-resume.md) · [失败实验索引](docs/decision-log/rejected-experiments.md)

## 项目状态

**公开工程记录** · **参考基线：Ventura 13.7.8** · **不提供可直接刷入的个人 EFI**

本项目公开的是**硬件适配说明、验证过的配置思路、实验时间线、失败原因、验收方法和风险边界**。它不会公开原机完整 EFI、SMBIOS 身份、序列号、MLB、ROM、UUID、MAC、SPI/EC 固件镜像或个人备份。

| 项目 | 本机最终记录 |
|---|---|
| 机型 | Xiaomi Mi Notebook Pro 15.6" TM1701 |
| CPU | Intel Core i7-8550U（Kaby Lake Refresh） |
| iGPU | Intel UHD Graphics 620 |
| dGPU | NVIDIA MX150；macOS 路径中禁用 |
| 内存 | 8 GB |
| 内屏 | BOE NV156FHM-N61，1920×1080 |
| 音频 | Realtek ALC298，AppleALC layout 30 |
| 触控板 | ETD2303，I²C/GPIO |
| 无线最终方案 | Broadcom BCM943602CS，macOS 原生 Wi-Fi / Bluetooth |
| OpenCore | 1.0.7 |
| 最终记录系统 | macOS Ventura 13.7.8（22H730） |
| 启动模型 | MacBookPro15,4（只记录型号，不公开任何身份字段） |

**注意：** TM1707 与 TM1701 同属 8 代 Kaby Lake Refresh 路线，但不同批次/主板/屏幕/无线卡仍须逐项核对。TM1905/TM1963 是 10 代 Comet Lake，不能照搬本项目的 ACPI、BIOS、USB 和电源结论。

## 先看这几份

1. **[从零安装指南](docs/02-installation.md)**：准备、BIOS、EFI、安装介质、首次启动、安装后验收。
2. **[BIOS 与 EC 记录](docs/03-bios-and-ec.md)**：DVMT 64 MB、CFG Lock、E07 BIOS、EC502 风扇策略，以及哪些固件动作不应照抄。
3. **[OpenCore 基线](docs/04-opencore-baseline.md)**：重要 quirks、boot-args、Vault、双系统启动。
4. **[ACPI 设计](docs/05-acpi-design.md)**：每张 SSDT/DMAR 的作用、依赖和保留理由。
5. **[Kext 清单](docs/06-kext-inventory.md)**：最终启用版本、未启用组件和失败升级。
6. **[S4 深度案例](docs/case-studies/s4-sleep-resume.md)**：整个项目最重要的工程案例，包含 0x27 panic、DMA/IOMMU、HibernationFixup、PXSX、LID/GPE、EC A/B，以及最后的 12h/3h 折中。
7. **[已接受的 Quirks 决策](docs/decision-log/accepted-quirks.md)**：哪些设置经过实机证明必须保留。
8. **[失败实验总表](docs/decision-log/rejected-experiments.md)**：不再重复踩坑。

## 最终体验边界

已经有实机证据支持的结果：

- Ventura 13.7.8 正常启动；OpenCore 隐藏 Picker，普通启动直接进入默认 macOS。
- UHD620 / Metal 3 正常；内屏真实 BOE EDID 保留，1424×802 HiDPI、Color LCD 色彩配置通过。
- HDMI 外接 4K30 正常；USB-C / DisplayPort 4K60 有独立验收记录。不要把 HDMI 与 USB-C/DP 的带宽上限混为一谈。
- ALC298 扬声器、耳机插拔、耳机麦克风和内置麦克风通过现场测试。
- I²C 触控板、PS/2 键盘和功能键按键映射通过实机验收。
- Broadcom 原生 Wi-Fi / Bluetooth 工作，AirDrop 文件互传通过。
- 两个 USB-C 数据口均达到 5 Gbps，USBMap 固化。
- 电池 S3 → 延迟 S4 的路径在 Sonoma 与 Ventura 都有真实记录；最终选择 Battery 高电量 12 小时、低电量 3 小时，`lowpowermode=0`；AC 保持普通睡眠。
- 深 S4 后**开盖本身不能唤醒**，需要短按电源键，再经 OpenCore 恢复原 macOS 会话。这是接受的硬件/固件边界，不伪装成原生 MacBook 行为。

尚未宣称完美或普遍适用的项目：SD 卡读卡器、所有外设/所有睡眠场景、不同屏幕批次、不同无线卡、不同 BIOS/EC 版本、长期跨季节热稳定性。

## 重要安全说明

- Hackintosh 是非 Apple 硬件运行 macOS 的非官方方案；Apple 不为此硬件组合提供支持。
- **不要直接使用别人的完整 EFI。** SMBIOS 序列号、MLB、ROM、UUID 必须每台机器独立生成；照抄会造成身份冲突和 Apple 服务问题。
- BIOS/EC/SPI 写入、DVMT/CFG Lock 修改和 EFI 替换均有变砖或无法启动风险。先备份、确认主板/芯片组、确认恢复路径；不要在远程无人值守状态下刷写。
- Windows EFI/BCD 与 macOS EFI 必须分开识别。NVMe 的 disk0/disk1 编号可能在启动后交换；**按硬盘型号、分区 UUID 识别，绝不固定盘号。**
- Ventura 13.7.8 是本项目冻结的已验收基线，不等于 2026 年新装机的安全推荐。它的 Apple 安全内容页面标注发布日期为 2025-08-20；部署前应重新评估仍受安全维护的 macOS 版本与应用兼容性。
- 本项目不发布 BIOS/EC 固件镜像、个人 EFI、原始 panic 中的个人标识、网络地址或私有备份。

## 证据等级

文档中的结论按以下等级区分：

- **PASS / 实机通过**：有用户物理测试或可重复的现场日志。
- **PASS / 配置校验**：`ocvalidate`、Vault、哈希、配置解析通过，但不等于物理功能通过。
- **DEGRADED / 可用但有瑕疵**：功能可用，仍存在已知体验缺陷。
- **FAIL / 实测失败**：真实 A/B 未通过，已回滚。
- **REJECTED / 不再采用**：有足够证据不再进入生产基线。
- **UNVERIFIED / 未验证**：只枚举、只推断、或没有完成物理测试。

## 项目结构

```text
docs/
  00-scope-and-evidence.md
  01-hardware-and-compatibility.md
  02-installation.md
  03-bios-and-ec.md
  04-opencore-baseline.md
  05-acpi-design.md
  06-kext-inventory.md
  07-graphics-display-hidpi.md
  08-audio-input-trackpad-keyboard.md
  09-wireless-bluetooth.md
  10-usb-storage-dualboot.md
  11-validation-matrix.md
  case-studies/s4-sleep-resume.md
  decision-log/accepted-quirks.md
  decision-log/rejected-experiments.md
  references.md
```

## 贡献

欢迎同款机型用户补充：主板/BIOS 版本、CPU、屏幕 EDID、无线卡、ACPI 差异、完整日志和可重复 A/B。请先读 [CONTRIBUTING](CONTRIBUTING.md) 与 [SECURITY](SECURITY.md)。

**本项目最重要的原则：每一个最终设置，都应能回答三个问题——改了什么、为什么改、什么证据证明它值得留下。**

