# Cadence Sigrity Python 命令参考

> [!IMPORTANT]
> 本 README 只收录了作者在实际使用中整理的部分笔记，并非完整的命令参考。要查阅各模块的命令及详细参数，请直接查看下方列出的六份模块文档。

本命令库基于 **Sigrity and Systems Analysis 2026.1 系列**，收录了从 Cadence Python 环境导出的六份模块文档。按模块查阅类、函数和参数：

- [`sigrity`](SigritySuite.sigrity.html)：提供所有 Layout Workbench 工具的基础函数。
- [`curvedb`](SigritySuite.curvedb.html)：BNP Viewer。
- [`mat`](SigritySuite.mat.html)：Material Manager。
- [`amm`](SigritySuite.amm.html)：Analysis Model Manager。
- [`diagram`](SigritySuite.diagram.html)：Diagram view manager。
- [`cmsa3d`](cmsa3d.html)：Clarity 3D Workbench 的 Python 命令模块。

本库不限于 PowerSI；具体接口是否可用，以目标工具及其安装版本为准。

## 命令合集的导出方式

所有模块都使用以下方式导出命令文档。在对应的 Cadence Python 环境中，将两处 `模块名` 替换为要导出的模块名称。

```python
import pydoc
import 模块名
pydoc.writedoc(模块名)
```

## 批处理命令行参数

下例从命令行启动 Broadband SPICE，在不打开图形界面的情况下运行 Python 脚本：

```cmd
BroadbandSPICE.exe -b -NoUI -py "example.py"
```

- `-b`：选择批处理模式（batch mode），适合从命令行自动执行任务。在 Broadband SPICE 中，它具体指批处理提取模式。
- `-NoUI`：不启动图形界面。在 Linux 控制台或计算集群上运行批处理时，通常需要此选项。
- `-py`：由工具执行后面指定的 Python 脚本。Broadband SPICE 文档也列出了 `*.py` 命令行选项。

各工具对 `-b` 的说明并不完全相同：

| 工具 | `-b` 的作用及省略时的行为 |
| --- | --- |
| Broadband SPICE | 以批处理提取模式运行。在 Passivity Mode IV 下，这是默认行为，可省略 `-b`。 |
| PowerSI | 进入批处理模式，使批处理仿真参数表中的其他开关生效。省略 `-b` 时，按 `.spd` 文件指定的模式仿真。 |
| XcitePI | `-b` 或 `/b` 均表示无 GUI 批处理，仿真结束后程序自动退出。省略时进入交互模式，仿真结束后程序不会自动退出。 |
| T2B | 以批处理模式运行。 |

XtractIM、OptimizePI 和 Translators 系列（如 SPDLinks、Pads2Spd、Dsn2Spd）的批处理命令也以 `-b` 开头。这不表示它们都支持 Broadband SPICE 示例中的整组参数，也不表示它们都会在任务完成后自动退出。

### PowerSI：直接仿真已有工程

据 2026.1 产品手册的检索摘要，`powersi.exe -b "工程.spd"` 不带脚本时会加载指定工程，按工程已有设置直接开始仿真，不会停下来等待设置。这不等于不能调整任何参数：有对应批处理开关的选项可以在启动时指定。如果需要先修改端口、替换模型，再启动求解，则应使用业务脚本明确执行顺序。

## 外部启动示例：以 PowerSI 为例

以下三种写法均以 Windows 下的 `powersi.exe -py 脚本路径` 为例：外部程序启动 PowerSI，业务脚本由 PowerSI 执行，不要求隐藏窗口。这一启动方式已在用户的 PowerSI 环境验证；`-py` 是该示例的启动参数，不是 `sigrity` 模块命令。用于其他 Cadence 工具时，请核对其可执行文件和启动参数，而不是只替换程序名。使用前将示例路径替换成实际文件路径。

## PowerShell：从终端启动

```powershell
$powerSI = 'C:\Example\Cadence\tools\bin\powersi.exe'
$script  = 'D:\Example\jobs\simulation.py'
& $powerSI -py $script
```

## 外部 Python：启动后立即返回

适合外部程序需要继续响应操作的情况。PID 标识刚启动的进程，不表示业务脚本已经执行成功。此写法默认继承外部程序的工作目录。

```python
import subprocess
from pathlib import Path

powersi = Path(r"C:\Example\Cadence\tools\bin\powersi.exe")
script = Path(r"D:\Example\jobs\simulation.py")

for path in (powersi, script):
    if not path.is_file():
        raise FileNotFoundError(path)

process = subprocess.Popen([str(powersi), "-py", str(script)])
print(f"已启动 PowerSI，进程 ID：{process.pid}")
```

## 外部 Python：等待启动的进程退出

适合串行任务。`cwd` 将子进程的初始工作目录设为脚本所在目录；这与是否等待无关，`Popen` 也可传入相同的 `cwd`。`check=True` 在进程以非零状态码退出时报错。即使脚本已返回，只要该进程未退出，`run` 就会继续等待。

```python
import subprocess
from pathlib import Path

powersi = Path(r"C:\Example\Cadence\tools\bin\powersi.exe")
script = Path(r"D:\Example\jobs\simulation.py")

for path in (powersi, script):
    if not path.is_file():
        raise FileNotFoundError(path)

subprocess.run(
    [str(powersi), "-py", str(script)],
    cwd=script.parent,
    check=True,
)
```

## PowerSI 能直接接收 Python 源码吗？

已验证的 `-py` 用法传入的是**脚本路径**。目前查到的 2026.1 资料没有给出类似 `python -c "代码"` 的 PowerSI 用法，也没有确认它能从标准输入执行 Python。不能仅把 `Popen` 中的路径换成源码字符串，或设置 `stdin=PIPE`，就指望 PowerSI 执行这些代码。

据手册检索摘要，GUI 的 Script IDE 可以直接输入并运行 Python；这是界面内交互，并非外部程序向已有窗口发送命令的接口。批处理的 `-recpython` 用于根据 Tcl 脚本生成 `.py` 文件，也不是执行内联 Python 的开关。

## 目录版 EXE：随包分发固定脚本，每次只传参数

如果每次只变工程路径、端口名或频率，不必为每个任务生成新的 `.py`。可以把固定流程写在 `bootstrap.py`，作为独立资源随外部启动器发布。假设发布目录中有 `MyApp.exe` 和 `scripts/bootstrap.py`：脚本必须位于 PowerSI 可读取的实际路径，仅将它作为 Python 模块封入 EXE 不够。

以下是集成示例，环境变量传参未在 PowerSI 中实测。打包后的外部启动器按 **EXE 所在目录**定位脚本，而不是按当前工作目录查找：

```python
import os
import subprocess
import sys
from pathlib import Path

powersi = Path(r"C:\Example\Cadence\tools\bin\powersi.exe")
bootstrap = Path(sys.executable).resolve().parent / "scripts" / "bootstrap.py"
layout = Path(r"D:\Example\jobs\board.spd")
for path in (powersi, bootstrap, layout):
    if not path.is_file():
        raise FileNotFoundError(path)

subprocess.Popen(
    [str(powersi), "-py", str(bootstrap)],
    env={**os.environ, "POWERSI_LAYOUT": str(layout)},
)
```

由 PowerSI 执行的 `scripts/bootstrap.py` 从同名变量取值，再把路径传给自己的工程加载流程。下例只演示取参，不会启动仿真：

```python
import os
from pathlib import Path

layout = Path(os.environ["POWERSI_LAYOUT"])
```

如果连执行步骤也随任务变化，外部程序可以生成任务专属 `.py`，再将文件路径交给 `-py`。另一种未实测的做法是让固定入口从环境变量读取并执行短小、可信的源码；环境变量有长度限制，且不能执行来自不可信输入的代码。这仍依赖入口文件，不是 PowerSI 原生的无文件执行方式。

## 动态任务：向指定工具传入 Python 代码，并收回步骤进度

如果每次只换参数，上一节的固定脚本就够了。如果连执行步骤也要按任务生成，又不想为每次运行写一个 `.py` 文件，可以让外部启动器和固定入口共用一条本机连接。这里的 `-py` 仍然只接收**入口文件的路径**；任务源码和进度消息走的是应用自定义协议，不是 Cadence 命令行参数或原生进度 API。

**启动器 → 指定工具：**启动新进程并加载入口；**入口 → 启动器：**回连；**启动器 → 入口：**发送源码；**入口 → 启动器：**确认接收，随后回报步骤和结束状态。

1. 启动器先在本机回环地址监听一个临时端口，为本次任务生成一次性密钥。启动所选 Cadence 可执行文件时，将端口和密钥加入子进程环境，命令行只传启动选项和固定入口的文件路径。打包时须把入口作为可读取的文件分发。这个流程不接管已打开的窗口，也不依赖 `-NoUI`。
2. 入口从环境中取出并移除连接参数，回连启动器。启动器用随机数和 HMAC-SHA256 校验回连方，再发送带长度和校验标签的 UTF-8 源码。入口先核验长度字段，再按长度收全并校验源码，成功后才回传接收确认。
3. 入口把进度和完成回调提供给任务，再在 Cadence 的 Python 环境中编译、执行源码。只有可信启动器生成的任务才应走这条通道：认证可以校验消息来源和完整性，不会加密源码，也不会限制代码在 Cadence 进程中的权限。

### 启动器看到的状态分别说明什么？

| 状态 | 可以确认 | 不能据此断定 |
| --- | --- | --- |
| 已启动（有 PID） | 操作系统已创建目标进程 | 入口已运行或源码已送达 |
| `R`：已接收 | 入口已收全并校验源码 | 编译成功或第一步已执行 |
| `P`：步骤进度 | 内部脚本已到达发出这条消息的位置 | 当前操作已完成，或求解器已达到某个百分比 |
| `E`：执行错误 | 入口捕获到编译或执行异常并回报摘要 | 之前的操作已经回滚 |
| `D`：任务完成标记 | 脚本走到了显式调用完成回调的位置 | GUI 已退出、异步求解已结束或结果文件正确 |

例如，任务只有「加载输入」和「导出结果」两步：先发 `P：已完成 0/2，正在加载输入`，加载调用返回后才发 `P：已完成 1/2，正在导出结果`；导出调用返回后发 `P：已完成 2/2`，再发 `D`。长调用期间没有新消息，不等于卡死。如果某一步只是提交异步作业，它返回也不等于作业完成，仍须检查求解状态和预期输出。若脚本还要请求关闭 GUI，`D` 在退出请求**之前**发送。

排查时看最后一个已确认的状态：进程在回连前退出，应先查启动参数和入口路径；等待回连超时，还应核对工具版本、许可证及 `-py` 支持情况。收到 `R` 后若连接中断且没有 `D` 或 `E`，只能记为「最终状态未知」，不能当作成功，也不能假定任务没有产生输出。

上述双向流程据应用源码整理，未在目标 Cadence 环境做端到端验证。前文已验证的是 Windows 下 PowerSI 通过 `-py` 加载脚本，并非整套回传协议或其他工具的兼容性。

## Sigrity 内部 Python：按脚本所在目录定位文件

如果 `board.spd` 和 Python 脚本放在同一目录，在 Sigrity 内部执行脚本时，可用 `sigrity.get_script_path()` 定位工程文件，不必假定启动 Cadence 时的工作目录就是脚本目录。

据 [2026.1 本地 API 文档：sigrity.get_script_path](SigritySuite.sigrity.html#-get_script_path)，即使以相对路径启动脚本，接口返回的也是**脚本所在目录的绝对路径**。不要直接对返回值调用 `os.path.dirname()`，否则会退到上一级目录。下例保留了防御性判断：仅当返回值指向已有文件时才取其父目录；这并非文档规定的另一种返回形式。

```python
import os

try:
    sigrity
except NameError:
    import sigrity

currentpath = sigrity.get_script_path()
if not isinstance(currentpath, str) or not os.path.isabs(currentpath):
    raise ValueError("Expected an absolute script path: %r" % (currentpath,))
if os.path.isfile(currentpath):
    _directory = os.path.dirname(currentpath)
elif os.path.isdir(currentpath):
    _directory = currentpath
else:
    raise ValueError("Script path does not exist: %r" % (currentpath,))

def local_file(name, directory=_directory):
    return os.path.normpath(os.path.join(directory, name))

layout_path = local_file("board.spd")
```

`local_file` 接收相对文件名或子路径。例如，脚本在 `D:\Example\jobs\simulation.py` 时，`layout_path` 为 `D:\Example\jobs\board.spd`。函数只拼接并规范化路径，不会创建文件，也不检查 `board.spd` 是否存在。

## PowerSI 内部 Python：保存或放弃更改后退出

以下语句用于已可访问 `sigrity` 的 PowerSI 内部脚本，不是外部启动器的命令。按需要选一条；如需保留窗口，不调用 `exit`。

- **保存更改并退出：** `sigrity.exit(True)`
- **不保存更改并退出：** `sigrity.exit(False)`（默认行为，未保存的更改会丢失）。

如果脚本提交的是异步仿真，应确认求解已完成、预期结果已写入，再退出。保存工程和进程返回零都不能替代对仿真结果的检查。退出参数依据：[本地 API 文档：sigrity.exit](SigritySuite.sigrity.html#-exit)。
