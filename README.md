# Cadence Sigrity Python 命令参考

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

## PowerSI 内部 Python：保存或放弃更改后退出

以下语句用于已可访问 `sigrity` 的 PowerSI 内部脚本，不是外部启动器的命令。按需要选一条；如需保留窗口，不调用 `exit`。

- **保存更改并退出：** `sigrity.exit(True)`
- **不保存更改并退出：** `sigrity.exit(False)`（默认行为，未保存的更改会丢失）。

如果脚本提交的是异步仿真，应确认求解已完成、预期结果已写入，再退出。保存工程和进程返回零都不能替代对仿真结果的检查。退出参数依据：[本地 API 文档：sigrity.exit](SigritySuite.sigrity.html#-exit)。
