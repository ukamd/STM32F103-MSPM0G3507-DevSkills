# Timer/PWM、ADC、DMA 与共享中断

## Timer / PWM

从 SysConfig 选实际存在且支持所需模式的 TIMA/TIMG 实例，核对时钟源、分频、计数位宽、load、计数方向与 CCP pinmux。按 TRM 对应模式计算周期，不能套 STM32 的 PSC/ARR 命名或固定加一公式。用该 SDK 的同实例官方例程确认 compare、输出动作、极性和停止/更新语义。

检查初始化后计数器是否真正启动；GPIO 输出、Timer 输出连接、通道及 advanced Timer 输出限制需同时成立。修改 duty 前确认 up/down/center 模式和 update latch，验证 0%/100% 边界。需精确时序时使用 Timer，不使用优化敏感的软件空循环。

## ADC

核对 ADC 实例、通道与实际模拟管脚、采样时间/源阻抗、分辨率、参考源与稳定时间。先单通道采集，再加入序列、事件触发、DMA；转换配置与 enable/start 的生命周期从当前生成代码和例程核对。Timer→事件路由→ADC→DMA 逐段验证，软件启动成功不证明硬件事件已连通。

ADC 电压范围按选定 VREF/VDDA，不能因其他 GPIO 有 5 V 容忍就接受超范围模拟输入。结果寄存器宽度、DMA 读写宽度、对齐、目标类型、序列长度一致。DMA 数据用静态或明确持久的缓冲区；处理完成通知后再消费对应数据区，避免处理尚在写入的区段。

## DMA：先确认通道能力

用户提供的 SLAU846A §4.2 将通道分为 basic/full-feature：重复传输模式只在支持该能力的通道可用。以本器件当前 TRM/SDK 查询实际通道能力和触发选择，不把任意通道当作 STM32 circular DMA。

记录：触发外设/事件、DMA channel、触发粒度（单元素/块）、source/destination 地址、增减/固定、数据宽度、长度、single/block/repeated、完成及错误 IRQ。UART 数据寄存器常固定、内存常递增，但仍按所选官方例程验证；ADC 序列可能涉及不同结果寄存器/触发策略，不能从 UART 配置复制。

配置、装载地址/计数后，按例程顺序启用外设 DMA 请求与通道，最后启动产生数据的源。优先确认最小传输，检查残余计数和 error，再加入重复模式。正常模式结束后按 SDK 明确重装；重复模式不自动提供应用级环形缓冲读写协议。

## 共享 IRQ、IIDX 与调试副作用

- Cortex-M0+ 具有四级可编程 IRQ 优先级，数字越小优先级越高；不套用 Cortex-M3 的 PRIGROUP/抢占与子优先级配置模型。
- 共享向量先识别 interrupt group 中的来源，再处理模块内部 pending。查当前 startup、device header 与生成的 IRQn，不能凭 GPIOA/B 名称生成 ISR 名。
- 在某些 MSPM0 模块中读取 IIDX 会确认/清除最高优先级 pending；用户手册 §4.2.10（DMA）和 §9（GPIO）说明这种副作用。调试器 watch/register 窗口持续读取也可能消耗事件。诊断先选择文档标明无副作用的状态寄存器，处理代码读一次 IIDX 后保存局部值。
- 按实际 IIDX/ICLR 语义处理，不能在 helper 已清除事件后再次读取期待同一值。共享源需服务所有已启用事件；避免无限循环阻塞其他来源。

回调/ISR 不进入时按“时钟/上电→外设请求→DMA 搬运/错误→pending→group/NVIC→真实向量→应用处理”定位。

依据：用户提供 SLAU846A §3.3、§4.2、§9；[最新版 TRM](https://www.ti.com/lit/pdf/SLAU846)；当前 SDK 示例和头文件。
