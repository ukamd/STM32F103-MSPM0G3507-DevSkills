# 双 Skill 安装与调用

本包提供两个独立、中文为主的 Skill：

| 目录 | 默认开发路线 |
| --- | --- |
| `stm32f103c8t6-dev/` | 通用 F103C8T6、传统 CubeMX、CubeF1 HAL、CMake/Ninja/GCC |
| `tmx-mspm0g3507-dev/` | 立创天猛星、CCS、TI Arm Clang、SysConfig、DriverLib |

每个目录含 `SKILL.md`、`agents/openai.yaml`、`references/` 与 MIT LICENSE。可以只安装其中一个；不依赖另一目录。不包含 SDK、工具程序、预制固件、用户提供的 PDF 或自动安装脚本。首版版本 0.1.0，资料核对日期 2026-11-01。

## 安装到项目

解压 ZIP，在包含两个 Skill 目录的文件夹执行以下 PowerShell。输入**已有目标工程**的根目录。仅复制所选 Skill 的完整目录，不只复制 `SKILL.md`；同名目录存在时停止，先比较版本。

```powershell
$projectDir = Read-Host '目标工程根目录的绝对路径'
if (-not (Test-Path -LiteralPath $projectDir -PathType Container)) { throw '目标工程不存在' }
$selectedSkills = @('stm32f103c8t6-dev','tmx-mspm0g3507-dev') # 按需只保留一个
$destination = Join-Path $projectDir '.agents/skills'
foreach ($name in $selectedSkills) {
    if (-not (Test-Path -LiteralPath (Join-Path (Join-Path $PWD $name) 'SKILL.md'))) { throw '请在解压后的包根目录运行' }
    if (Test-Path -LiteralPath (Join-Path $destination $name)) { throw "已有 $name，请先比较版本" }
}
New-Item -ItemType Directory -Path $destination -Force | Out-Null
foreach ($name in $selectedSkills) {
    Copy-Item -LiteralPath (Join-Path $PWD $name) -Destination $destination -Recurse
}
```

Linux/macOS 将完整目录复制到工程 `.agents/skills/`，同名已存在时先比较，避免覆盖。用户级安装可按客户端约定复制到 `$CODEX_HOME/skills`，未设置时通常为 `~/.codex/skills`；项目级与用户级选一种，避免重复发现。本次交付没有执行这些安装操作。

重新加载客户端或开启新会话，确认发现所安装的 Skill。其他支持标准 `SKILL.md` 的客户端按自身目录规则安装，`agents/openai.yaml` 是否展示取决于客户端支持。

## 调用示例

```text
使用 $stm32f103c8t6-dev 检查当前 F103C8T6 工程的 CMake 构建环境。
使用 $stm32f103c8t6-dev 将 USART1 改为 DMA 接收，保留现有应用代码。
使用 $stm32f103c8t6-dev 排查这个 ELF 的 SRAM 超限，不更改芯片容量。

使用 $tmx-mspm0g3507-dev 从匹配 SDK 的裸机例程创建天猛星 CCS 工程。
使用 $tmx-mspm0g3507-dev 配置板载 LED 和按键，并核对当前 PCB 接线。
使用 $tmx-mspm0g3507-dev 排查 UART0 DMA 收到第一帧后停止的问题。
```

两个 Skill 均允许自动发现，也可显式点名。提供现有工程、实际接线/原理图、SDK 版本与错误日志可减少反复确认。仅涉及软件构建时不必先提供探针；涉及下载再确定探针和硬件。

## 验证与来源

详见 [VALIDATION.md](VALIDATION.md)：格式、独立性与 ZIP 检查已做，场景为人工流程审查；未完成硬件测试或固件交叉编译。每个 Skill 的 `references/sources.md` 附一手资料及版本说明。天猛星参考资料已吸收用户提供 SLAU846A Rev. A 的相关约束；完整手册未随包分发。

结构参考 [STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills/tree/82e8bff032c95eae85592d82dec5dfac8fa50454)，保留上游 MIT 声明。ST/TI/Arm/FreeRTOS 的 SDK、手册与商标各自归原权利人，本包不重新许可这些第三方资料。
