# BOM 与器件选型初版

## 1. 选型原则

1. 高压/大电流器件优先满足耐压、电流、SOA、热阻、封装可制造性。
2. 安全链路器件优先满足失效安全、默认关断、可诊断。
3. MCU 先按外设资源和封装引脚数选，不按最低成本选。
4. 通信收发器必须匹配协议，CAN FD 不用传统低速/Classic-only 器件。
5. 所有外部接口必须有 ESD、限流、滤波或电平转换策略。
6. 本表候选器件不等于最终 BOM，缺关键参数时标 `TBD`。

## 2. 参考资料

以下只列官方/原厂资料入口，用于后续核对：

| 类别 | 资料 |
|---|---|
| STM32H743/753 | https://www.st.com/en/microcontrollers-microprocessors/stm32h743-753.html |
| STM32G431 | https://www.st.com/en/microcontrollers-microprocessors/stm32g431rb.html |
| STM32G474 | https://www.st.com/en/microcontrollers-microprocessors/stm32g474re.html |
| STM32G4 Motor Control | https://www.st.com/content/st_com/en/ecosystems/stm32-motor-control-ecosystem.html |
| ESP32-S3-WROOM-1 | https://www.espressif.com/en/module/esp32-s3-wroom-1-en |
| TCAN1042-Q1 | https://www.ti.com/product/TCAN1042-Q1 |
| ISO1042 | https://www.ti.com/product/ISO1042 |
| DRV8353 | https://www.ti.com/product/DRV8353 |
| SC16IS752/762 | https://www.nxp.com/products/interfaces/ic-spi-i3c-interface-devices/bridges/dual-uart-with-ic-bus-spi-interface-64-bs-of-transmit-and-receive-fifos-irda-sir-built-in-support%3ASC16IS752_SC16IS762 |

## 3. 核心器件候选

### 3.1 H7 主控

| 项目 | 要求 | 候选方向 | 状态 | 说明 |
|---|---|---|---|---|
| 主控 MCU | Ethernet、USB Type-C、FDCAN x2、多 SPI/UART、PWM/ADC、足够 GPIO | STM32H743/753 系列 | 待封装确认 | 官方资料显示 STM32H743/753 为 Cortex-M7 480 MHz 级别，适合作为主控候选 |
| 封装 | 高 IO 数 | LQFP176/BGA/TBD | TBD | Ethernet + USB Type-C + 多 UART/SPI + 仓门电机，LQFP144 可能紧 |
| 外部晶振 | HSE + LSE/TBD | TBD | TBD | Ethernet/USB 对时钟精度敏感 |
| 存储 | 参数/日志 | QSPI Flash 或 EEPROM/TBD | TBD | 是否需要离线日志由用户确认 |

当前建议：

- 优先按 STM32H743/753 高引脚封装做资源评估。
- 不在确认引脚预算前选小封装。

### 3.2 G4 电机控制 MCU

| 项目 | 要求 | 候选方向 | 状态 | 说明 |
|---|---|---|---|---|
| 前轮 G4 | 双路 FOC、CAN FD、ADC、比较器、互补 PWM | STM32G4 高资源型号，如 G474/G491 方向 | TBD | 双电机资源压力大，不建议只按低配 G431 定死 |
| 后轮 G4 | 同前轮 | STM32G4 高资源型号 | TBD | 与前轮复用设计 |
| 转向 G4 | 双路位置 FOC + 2 路 SPI 绝对编码器 | STM32G4 高资源型号 | TBD | 编码器接口和限位 GPIO 更多 |

当前建议：

- 先做 G474/G491 级别资源预算，再看能否降级。
- 不能只因为 STM32G431 常见就直接定为双 FOC 主控，必须核对 PWM/ADC/COMP/DMA/PinMux。

### 3.3 ESP32-S3

| 项目 | 要求 | 候选方向 | 状态 | 说明 |
|---|---|---|---|---|
| 无线模块 | Wi-Fi AP + 蓝牙，板载模块 | ESP32-S3-WROOM-1 或带外接天线版本 | 待天线方案确认 | PCB 天线版要求天线净空；金属外壳可能需要外接天线版本 |
| 通信 | 与 H7 通信 | UART 默认，SPI 可选 | 未冻结 | H7 侧保留 UART，SPI 看吞吐需求 |

必须确认：

- 外壳是否金属。
- 天线是否需要引出。
- 无线距离要求。

## 4. 电机驱动功率器件

### 4.1 Gate Driver

| 用途 | 要求 | 候选方向 | 状态 | 说明 |
|---|---|---|---|---|
| 轮毂三相驱动 | 24-48 V 系统，建议 75/100 V 余量，支持保护 | TI DRV8353 方向 | 待评估 | 官方资料显示 DRV8353 为 102 V max 三相智能 Gate Driver，适合 48 V 余量方向 |
| 转向三相驱动 | 同上，电流可能较小 | DRV8353 或低功率等效器件 | TBD | 根据转向电机电流可降规格 |
| 仓门三相驱动 | 5-24 V，建议 40/60 V 器件 | 低压三相 Gate Driver/TBD | TBD | 若电流小，可选更集成器件 |

注意：

- 每个三相电机通常需要一套三相 Gate Driver。
- 本板共有 7 个无刷电机通道，Gate Driver 数量和成本/面积会很大。
- 若选择集成驱动器，需要重新核对电流、散热和封装。

### 4.2 MOSFET

| 用途 | 要求 | 当前状态 |
|---|---|---|
| 轮毂 MOS | N 沟道，建议 75/100 V，低 Rds(on)，足够 SOA，适合散热 | TBD |
| 转向 MOS | 由峰值电流决定，耐压跟随主母线 | TBD |
| 仓门 MOS | 40/60 V 方向，电流按电机定 | TBD |
| 锁 MOS | 40/60 V 方向，按锁峰值电流定 | TBD |

不能确定的原因：

- 轮毂峰值相电流未知。
- 持续电流和散热条件未知。
- 板子铜厚和散热器未知。
- 允许温升未知。

选型时必须校核：

- Vds 额定值。
- Id 连续/脉冲能力。
- Rds(on) 在实际 Vgs 和温度下的值。
- Qg 与驱动能力。
- SOA。
- 封装热阻。
- 反向恢复/体二极管特性。
- 并联需求。

### 4.3 电流采样

| 方案 | 状态 | 说明 |
|---|---|---|
| 三低边采样 | 推荐方向 | 成本可控，FOC 常用，但布局和采样窗口要做好 |
| 三相采样 | 推荐方向 | 性能好，成本更高 |
| 单母线采样 | 不作为主方案 | 不符合原始需求中 FOC 采样建议 |

待定：

- 采样电阻阻值。
- 采样功率。
- 放大器增益。
- ADC 满量程。
- 过流阈值。

## 5. 通信器件

### 5.1 CAN FD

| 用途 | 候选方向 | 状态 | 说明 |
|---|---|---|---|
| 内部电机 CAN FD | TCAN1042-Q1 或同类 CAN FD 收发器 | 候选 | 官方资料标明支持 CAN FD，并有车规版本 |
| 外部隔离 CAN FD | ISO1042 或同类隔离 CAN FD 收发器 | 候选 | 官方资料标明 ISO1042 支持隔离 CAN FD，并有较高总线故障保护 |

设计要求：

- 每路 CANH/CANL 加 TVS。
- 外部 CAN 加共模电感。
- 终端 120R 跳帽。
- 隔离 CAN 需要隔离电源。

### 5.2 UART 扩展

| 用途 | 候选方向 | 状态 | 说明 |
|---|---|---|---|
| 8 路超声波 UART | NXP SC16IS752/762，4 颗双 UART，或八通道 UART 桥 TBD | 候选 | 官方资料显示 SC16IS752/762 是 SPI/I2C 到双 UART，带 FIFO |

需要确认：

- 超声波模块波特率。
- 是否需要同时接收 8 路。
- 允许延迟。
- 每路是否需要独立电源开关。
- SPI 片选和中断引脚数量。

### 5.3 RS485

| 项目 | 要求 | 状态 |
|---|---|---|
| RS485 收发器 | 3.3 V 逻辑，半双工，ESD 强，故障保护 | TBD |
| 终端 | 120R 跳帽 | 必须 |
| 偏置 | 上拉/下拉可选 | 推荐 |
| 隔离 | 是否隔离 TBD | 由外部线长/地电位决定 |

## 6. 电源器件

### 6.1 主电源保护

| 功能 | 候选方向 | 状态 |
|---|---|---|
| 总保险 | 汽车保险/大电流保险/TBD | 待电流确定 |
| 反接保护 | 理想二极管控制器 + MOS，或接触器前级保护/TBD | 待系统要求 |
| 预充 | 预充电阻 + MOS/继电器 + 检测 | 必须 |
| 主接触器 | 外置还是板载驱动 TBD | 待机械/电流确认 |
| TVS | 按电池最高电压和浪涌选 | TBD |
| 泄放电阻 | 刹车回灌/母线泄放预留 | 必须预留，参数 TBD |

### 6.2 DC/DC

| 电源轨 | 要求 | 状态 |
|---|---|---|
| 48 V -> 24 V | 给 V_DOOR/V_LOCK/外设，电流 TBD | 待电流预算 |
| 48 V -> 12/15 V | 给 Gate Driver | 待 Gate Driver 和 MOS 方案 |
| 12/15 V -> 5 V | 给 Hall、外设、逻辑前级 | 待电流预算 |
| 5 V -> 3.3 V | 给 MCU、ESP32、传感器 | 待电流预算 |

不能现在定型号的原因：

- V_DOOR/V_LOCK 是否由板上 24 V 供电未确认。
- 外设电流未知。
- ESP32 峰值电流预算未冻结。
- Gate Driver 数量和栅极电荷未知。

## 7. 接口保护器件

| 接口 | 保护 |
|---|---|
| CAN | TVS、共模电感、终端跳帽，外部 CAN 可隔离 |
| RS485 | TVS、终端、偏置、可选隔离 |
| USB Type-C | Type-C 连接器、CC 下拉、USB 专用低电容 ESD、VBUS 检测/保护 |
| Ethernet 网口 | PHY、磁性器件/RJ45、ESD、浪涌策略、屏蔽接地 |
| UART | 串阻、ESD、双电源电平转换 |
| Hall | 上拉、滤波、ESD、电平保护 |
| ABI 编码器 | 串阻、ESD、上拉预留、电平转换 |
| SPI 编码器 | 串阻、ESD、3.3 V 滤波 |
| 限位开关 | RC/软件滤波、ESD、上拉/下拉、故障检测 |

## 8. BOM 未决项

| 项目 | 为什么必须确认 |
|---|---|
| 轮毂峰值电流 | 决定 MOS、采样电阻、保险、铜厚、散热 |
| 转向电机参数 | 决定转向驱动器、MOS、采样范围 |
| 仓门电机参数 | 决定是否 H7 直控可行 |
| 锁类型和电流 | 决定 MOS/继电器、续流、保护 |
| 板框/散热 | 决定 MOS 封装和铜皮策略 |
| 连接器 | 决定 PCB 边界、线束、耐流、安全间距 |
| 外壳材质 | 决定 ESP32 天线版本 |
| 生产方式 | 决定封装限制、层数、铜厚、成本 |
