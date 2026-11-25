# 项目记忆

## 项目概况与现状

本命令库基于 Sigrity and Systems Analysis 2026.1 系列，整理 Cadence Python 模块命令，供其他项目查阅，不限于 PowerSI。外部启动示例以 PowerSI 为例，不据此推断其他工具支持相同参数。用户已确认在其 Windows 环境中，外部程序以 `powersi.exe -py 脚本路径` 启动 PowerSI 后，业务脚本能在 Cadence 内部环境运行。此结论不代表所有版本，也不表示可以接管已有窗口。

## 资料与文件地图

- [README](README.md)：仓库概览、模块用途索引、模块文档导出方法、各工具的批处理命令行参数，以及 PowerSI 外部启动和内部退出示例。批处理参数说明依据用户提供的工具文档摘录，不属于已实测的 PowerSI 启动行为。
- `SigritySuite.<模块名>.html`：Cadence Python 环境导出的模块 API 文档，现收录 `sigrity`、`curvedb`、`mat`、`amm` 和 `diagram`。其中 [sigrity.exit](SigritySuite.sigrity.html#-exit) 说明保存参数和默认值；退出行为按文档记录，未在此次整理中实测。

- [`cmsa3d.html`](cmsa3d.html)：Clarity 3D Workbench 的 Python 命令模块文档。导出方式与其他模块相同。

## 规则与限制

- 保留用户提供的示例命令原文；如需改写，先征得用户同意。

- 区分进程启动、进程退出和仿真成功。异步求解任务须确认求解完成并写入结果，再按需要保存或退出。
- 显示或隐藏 UI 不是运行内部 Python 脚本的前提；参考示例不强制隐藏窗口。
