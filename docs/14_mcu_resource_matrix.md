# MCU 资源矩阵

## 1. 状态

当前未冻结：

- STM32H7 具体型号。
- STM32H7 封装。
- STM32G4 具体型号。
- STM32G4 封装。
- Gate Driver 是否需要 SPI。
- 仓门是否需要 FOC。
- 以太网 PHY/RJ45 方案。

因此本文档只做资源需求矩阵，不做最终 PinMap。

## 2. H7 外设需求矩阵

| 功能 | 外设类型 | 数量 | 强制性 | 风险 | 备注 |
|---|---|---:|---|---|---|
| 内部电机 CAN FD | FDCAN | 1 | 必须 | 中 | 连接 3 个 G4 |
| 外部 CAN FD | FDCAN | 1 | 必须 | 中 | 建议隔离 |
| USB Type-C | USB FS/HS | 1 | 必须 | 中 | 默认 FS Device/CDC，是否 HS 待定 |
| Ethernet 网口 | ETH MAC | 1 | 必须 | 高 | 必须保留，会占用大量引脚 |
| ESP32 通信 | UART | 1 | 必须 | 低 | SPI 可选 |
| RS485 | UART + DE GPIO | 1 | 必须 | 低 | 半双工默认 |
| 调试 UART | UART | 0-1 | 可选 | 低 | USB Type-C CDC 为主 |
| 扩展 UART | UART | 4 | 必须 | 高 | 优先 H7 原生 UART |
| 8 路超声波 | SPI + CS/IRQ | 1 SPI | 必须 | 中 | 通过 SPI-UART 扩展 |
| IMU | SPI | 1 | 必须/待确认 | 低 | I2C 备选 |
| 仓门 PWM | 高级定时器 | 1 套 | 必须/待确认 | 高 | 若做三相互补 PWM |
| 仓门电流采样 | ADC/COMP | 1 套 | 必须/待确认 | 高 | 取决于控制方式 |
| 电子锁 | GPIO/PWM | 1-3 | 必须 | 低 | 控制、状态、过流 |
| 急停输入 | GPIO/EXTI | 2 | 必须 | 低 | 同时进硬件链 |
| 接触器控制 | GPIO | 1-2 | 必须 | 中 | 驱动和反馈 |
| 预充控制 | GPIO/ADC | 2-4 | 必须/待确认 | 中 | 取决于拓扑 |
| 电源检测 | ADC | 4-8 | 必须 | 中 | VBAT/24/15/5/3V3 等 |
| 外设电源控制 | GPIO | TBD | 可选 | 中 | 每路开关与否待定 |
| LED/蜂鸣器 | GPIO/PWM | TBD | 可选 | 低 | 用户体验/调试 |

## 3. H7 引脚压力判断

### 3.1 高压力来源

- Ethernet 网口。
- USB Type-C。
- 多路原生 UART。
- 仓门三相控制。
- 外部接口保护/控制/状态回读。
- 电源安全控制。

### 3.2 选型建议

在未做 PinMux 之前：

- 不建议选择低引脚数 H7。
- LQFP144 可能紧张。
- LQFP176 或更高引脚数封装优先评估。
- 如果不需要 H7 直控仓门 FOC，H7 压力会明显下降。

### 3.3 可能的降压策略

若 H7 资源不足：

| 方案 | 影响 |
|---|---|
| 仓门改独立小 MCU/智能驱动 | H7 释放 PWM/ADC/保护资源 |
| 4 路扩展 UART 也走 UART 桥 | H7 释放 UART 引脚 |
| Ethernet 不再作为裁剪项 | 用户已明确要求必须有网口；若资源紧张，应提升 H7 封装或裁剪其他非安全功能 |
| ESP32 改 SPI 或共享调试通道 | 视协议复杂度 |
| 外设电源不做每路独立控制 | 释放 GPIO，但诊断能力下降 |

## 4. 前/后轮 G4 资源矩阵

每个 G4 控制两个轮毂。

| 功能 | 每电机 | 双电机合计 | 资源类型 | 强制性 | 风险 |
|---|---:|---:|---|---|---|
| 三相互补 PWM | 6 | 12 | 高级定时器 | 必须 | 高 |
| PWM Break | 1 | 2 | Timer Break/COMP | 必须 | 高 |
| 电流采样 ADC | 3 | 6 | ADC 同步采样 | 必须 | 高 |
| 硬件过流 | 1-3 | 2-6 | COMP/Driver | 必须 | 高 |
| Hall | 3 | 6 | GPIO/Timer input | 必须 | 中 |
| ABI 编码器预留 | 3 | 6 | Timer encoder/GPIO | 建议 | 中 |
| SPI 编码器预留 | 4-6 | 8-12 | SPI/GPIO | 可选 | 中 |
| 母线电压 | 共享 1 | 1 | ADC | 必须 | 低 |
| 温度 | 1-2 | 2-4 | ADC | 必须 | 中 |
| CAN FD | 1 | 1 | FDCAN | 必须 | 低 |
| SWD/NRST/BOOT | 1 套 | 1 套 | Debug | 必须 | 低 |
| Driver SPI | 0-1 | 0-2 | SPI/GPIO | 取决于 Driver | 中 |

## 5. 转向 G4 资源矩阵

| 功能 | 每转向 | 双转向合计 | 资源类型 | 强制性 | 风险 |
|---|---:|---:|---|---|---|
| 三相互补 PWM | 6 | 12 | 高级定时器 | 必须 | 高 |
| PWM Break | 1 | 2 | Timer Break/COMP | 必须 | 高 |
| 电流采样 ADC | 3 | 6 | ADC 同步采样 | 必须 | 高 |
| 输出轴 SPI 编码器 | 1 | 2 | SPI + CS | 必须 | 高 |
| Hall/电机反馈 | 3 | 6 | GPIO/Timer input | TBD | 中 |
| 机械限位 | 2 | 4 | GPIO/EXTI | 必须 | 低 |
| 母线电压 | 共享 1 | 1 | ADC | 必须 | 低 |
| 温度 | 1-2 | 2-4 | ADC | 必须 | 中 |
| CAN FD | 1 | 1 | FDCAN | 必须 | 低 |
| SWD/NRST/BOOT | 1 套 | 1 套 | Debug | 必须 | 低 |
| Driver SPI | 0-1 | 0-2 | SPI/GPIO | 取决于 Driver | 中 |

## 6. G4 选型风险

必须逐项核对：

- 高级定时器能否同时输出 12 路互补 PWM。
- ADC 是否支持双电机同步采样。
- COMP/OPAMP 数量是否满足电流采样和过流保护。
- FDCAN 是否可用。
- PinMux 是否允许 PWM、ADC、CAN、SPI、Hall 同时布开。
- 封装引脚数是否足够。
- 是否需要外部运放，还是使用片上 OPAMP。

不能做的事：

- 只按“STM32G4 能做电机控制”就定型号。
- 只看 IO 数，不看 Timer/ADC/PinMux。
- 在原理图后期才发现 FOC 双通道资源冲突。

## 7. H7-G4 信号关系

| 信号/总线 | H7 | G4 | 说明 |
|---|---|---|---|
| 内部 CAN FD | Master | Node | 命令/状态/故障 |
| `DRV_EN_*` | 控制/安全链 | 输入 | 硬件链优先 |
| `FAULT_*` | 输入 | 输出 | 驱动区故障 |
| `ESTOP_STATUS` | 输入 | 输入/硬件 | 急停状态 |
| SWD | 调试器 | 各 MCU | 独立接口 |

## 8. 未来 PinMap 表模板

确认型号后为每个 MCU 建表：

| MCU | Package Pin | GPIO | Signal | Peripheral | Direction | Reset State | Voltage | Pull | Safety Critical | Notes |
|---|---|---|---|---|---|---|---|---|---|---|

硬规则：

- 所有驱动 Enable 在复位态必须禁用。
- 所有 PWM 复位态不得打开 Gate Driver。
- 所有外部输入不得悬空。
- BOOT 配置不能让量产板误进 Bootloader，除非有明确升级方案。
- SWD 不要与关键运行信号冲突。

## 9. 需要用户确认

1. Ethernet PHY 选型、RMII/MII、RJ45 是否带磁/带灯。
2. USB Type-C 是否只用 FS Device/CDC，还是需要 HS。
3. 仓门是否必须 H7 三相直控？
4. G4 是否接受 BGA/QFN，还是必须 LQFP 便于焊接？
5. 是否要求所有 MCU 都能从 H7 远程升级？
