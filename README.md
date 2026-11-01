# STM32F103-MSPM0G3507-DevSkills

面向 **STM32F103C8T6** 与 **立创天猛星 MSPM0G3507** 的两个独立 AI 开发 Skill，覆盖环境定位、工程创建、外设配置、固件开发、构建、烧录与调试。

本仓库提供中文原版，文本采用 UTF-8，目录名和接口标识保留英文。每个 Skill 使用简短入口与按任务查阅的参考资料，可以单独安装。

## 两个 Skill

| Skill | 目标 | 新工程默认开发路线 |
| --- | --- | --- |
| [stm32f103c8t6-dev](stm32f103c8t6-dev/SKILL.md) | 通用 STM32F103C8T6 板卡 | 传统 CubeMX、STM32CubeF1 HAL、CMake、Ninja、Arm GNU Toolchain |
| [tmx-mspm0g3507-dev](tmx-mspm0g3507-dev/SKILL.md) | 立创天猛星 MSPM0G3507 | CCS、TI Arm Clang、MSPM0 SDK、SysConfig、DriverLib |

默认裸机开发，FreeRTOS 按需启用。接手已有工程时保留原驱动和构建体系，不自动迁移工具链。STM32 部分不预设 LED、晶振与接线；天猛星提供带来源及核实状态的板级参考。

## 安装

克隆或下载本仓库，将需要的完整 Skill 目录复制到目标工程的 `.agents/skills/`，或客户端支持的用户级 Skill 目录。项目级与用户级选择一种，避免重复发现；已有同名目录时先比较版本。

```text
your-project/
  .agents/
    skills/
      stm32f103c8t6-dev/
        SKILL.md
        agents/
        references/
        LICENSE
      tmx-mspm0g3507-dev/
        SKILL.md
        agents/
        references/
        LICENSE
```

每个目录必须完整复制，不只复制 `SKILL.md`。两个 Skill 可独立使用，无需同时安装。重新加载客户端或开启新会话后确认发现。

具体安装命令与更多调用示例见 [INSTALL.md](INSTALL.md)。

## 调用示例

```text
使用 $stm32f103c8t6-dev 检查当前 F103C8T6 工程的 CMake 构建环境。
使用 $stm32f103c8t6-dev 配置 USART1 DMA 接收并保留现有业务代码。
使用 $stm32f103c8t6-dev 排查 SRAM 超限，不更改芯片容量。

使用 $tmx-mspm0g3507-dev 从匹配 SDK 的裸机例程创建天猛星 CCS 工程。
使用 $tmx-mspm0g3507-dev 核对板载 LED、按键和当前 PCB 接线。
使用 $tmx-mspm0g3507-dev 排查 UART0 DMA 收到第一帧后停止的问题。
```

两个 Skill 保留自动发现，也可显式点名。实际命令和 API 以项目锁定版本的头文件、本机帮助及官方文档为准。

## 验证范围

已进行 Skill 格式、YAML、引用完整性、独立安装引用及打包一致性检查；场景验证为人工工作流程审查。**尚未进行固件交叉编译、烧录或实板测试。** 详细记录见 [VALIDATION.md](VALIDATION.md)。

天猛星的相关约束参考了用户提供的 TI SLAU846A Rev. A 手册，完整 PDF 不随仓库分发。SDK、开发工具、探针、原理图和固件产物也不随仓库分发；使用时需结合实际硬件和工具版本。

## 来源与许可

能力划分与资料组织参考 [keysking/STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills/tree/82e8bff032c95eae85592d82dec5dfac8fa50454)，具体技术内容分别针对 STM32F1 和 TI MSPM0 重新编写。

本仓库保留上游 [MIT 许可证](LICENSE)，两个 Skill 目录也各自附带 LICENSE，便于独立分发。ST、TI、Arm、FreeRTOS 的文档、SDK、工具及商标各自归其权利人；本仓库不重新许可这些第三方资料。

一手资料及版本边界见各 Skill 的来源页：[STM32](stm32f103c8t6-dev/references/sources.md)、[天猛星](tmx-mspm0g3507-dev/references/sources.md)。
