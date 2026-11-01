# 来源、版本与许可

本技能为中文开发工作流，参考 STM32C5DevSkills 的七阶段能力划分并针对 F103 重写。参考基线：[keysking/STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills/tree/82e8bff032c95eae85592d82dec5dfac8fa50454)。保留其 MIT 声明于本目录 LICENSE；厂商手册、SDK 和工具不随包分发。

核对日期：2026-11-01。本文的 CLI 例子是条件化参考，不宣称所有工具版本均已实测。版本升级时先读取本机手册与 help，再修改工程。

| 内容 | 一手依据 |
| --- | --- |
| 容量、封装、电气和引脚 | [F103C8 数据手册](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf) |
| RCC、AFIO、DMA、定时器及 ADC | [RM0008](https://www.st.com/resource/en/reference_manual/cd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) |
| 传统 CubeMX 脚本及工程生成 | [UM1718](https://www.st.com/resource/en/user_manual/um1718-stm32cubemx-for-stm32-configuration-and-initialization-c-code-generation-stmicroelectronics.pdf) |
| HAL1/LL 与 CMSIS | [STM32CubeF1](https://github.com/STMicroelectronics/STM32CubeF1)、[F1 HAL](https://github.com/STMicroelectronics/stm32f1xx-hal-driver)；实际工程版本优先 |
| 构建命令 | [CMake](https://cmake.org/cmake/help/latest/manual/cmake.1.html)、[Arm GNU](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain) |
| 下载与异常 | [CubeProgrammer](https://dev.st.com/stm32cube-docs/prog/latest/en/index.html)、[PM0056](https://www.st.com/resource/en/programming_manual/cd00228163-stm32f10xxx20xxx21xxl1xxxx-cortexm3-programming-manual-stmicroelectronics.pdf) |

编写时对本机 STM32CubeF1 1.8.7 头文件核对了 ReceiveToIdle DMA 的声明；不据此要求其他工程升级或假定旧版本已有该 API。未捆绑可直接烧录的固件。
