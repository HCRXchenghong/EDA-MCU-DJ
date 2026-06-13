# 接口与引脚资源预算

## 1. 文档原则

当前尚未选定 STM32H7/STM32G4 的具体型号和封装，因此本文档是资源预算，不是最终引脚分配。

规则：

- 不给未确认封装强行分配具体管脚。
- 先统计外设数量、时序要求、保护要求。
- 等型号/封装确认后，再输出 `H7_PinMap.xlsx/md` 和 `G4_PinMap.xlsx/md`。

## 2. 命名约定建议

### 2.1 电源网

| 网名 | 含义 |
|---|---|
| `VBAT` | 电池输入主电源 |
| `VMOT` | 电机主母线，若与 VBAT 不分离可短接 |
| `24V_SYS` | 系统 24 V |
| `15V_DRV` | Gate Driver 15 V，若选 12 V 则改为 `12V_DRV` |
| `5V_SYS` | 系统 5 V |
| `3V3_SYS` | 系统 3.3 V |
| `3V3_A` | 模拟/编码器低噪声 3.3 V |
| `V_DOOR` | 仓门电机电源 |
| `V_LOCK` | 电子锁电源 |

### 2.2 信号前缀

| 前缀 | 含义 |
|---|---|
| `H7_` | H7 本地信号 |
| `G4F_` | 前轮 G4 |
| `G4R_` | 后轮 G4 |
| `G4S_` | 转向 G4 |
| `FL_` | 前左轮 |
| `FR_` | 前右轮 |
| `RL_` | 后左轮 |
| `RR_` | 后右轮 |
| `STF_` | 前转向 |
| `STR_` | 后转向 |
| `DOOR_` | 仓门 |
| `LOCK_` | 电子锁 |
| `ESTOP_` | 急停 |

## 3. STM32H7 外设资源预算

### 3.1 高优先级外设

| 功能 | 需求 | 数量 | 资源类型 | 备注 |
|---|---:|---:|---|---|
| 内部电机 CAN FD | H7 与 3 个 G4 通信 | 1 | FDCAN | CAN1 |
| 外部 CAN FD | 上位机/扩展 | 1 | FDCAN | CAN2，建议隔离 |
| Ethernet 网口 | 工控机/ROS/交换机/上位机 | 1 | ETH MAC | 必须保留，默认 RMII 方向，PHY/RJ45 待选型 |
| USB Type-C | 调试/升级/上位机/USB CDC | 1 | USB FS 或 HS | 必须保留，默认 USB 2.0 FS Device，HS/ULPI 待用户确认 |
| ESP32 通信 | H7 与 ESP32 | 1 | UART，SPI 可选 | 默认 UART |
| RS485 | 外部总线 | 1 | UART + DE GPIO | 半双工默认 |
| UART 扩展芯片 | 8 路超声波 | 1 | SPI + CS/IRQ/RST | 多颗桥接芯片共享 SPI |
| IMU | 姿态/运动 | 1 | SPI 优先 | I2C 备选 |
| 仓门电机 PWM | 三相驱动 | 1 | 高级定时器/TIM | 若做 FOC/六步需要互补 PWM |
| 仓门电流采样 | 电流闭环/保护 | 1 套 | ADC/COMP | 采样方式待定 |
| 急停检测 | 安全输入 | 2 | GPIO/EXTI | 同时进入硬件链路 |
| 主接触器控制 | 电源安全 | 1 | GPIO | 需驱动级 |
| 电子锁控制 | 锁驱动 | 1 | GPIO/PWM 可选 | 低边 MOS/继电器 |

### 3.2 H7 GPIO 预算

| 类别 | 估计数量 | 说明 |
|---|---:|---|
| 急停输入 | 2 | `ESTOP_IN1/2` |
| 驱动区 Enable 控制/反馈 | 3-6 | 前轮、后轮、转向，可含反馈 |
| 驱动区 Fault 输入 | 3-6 | 每个 G4/驱动区至少 1 |
| 主接触器控制/反馈 | 2-3 | Coil control + aux feedback |
| 预充控制/反馈 | 2-3 | 取决于预充拓扑 |
| 电子锁控制/状态 | 2-4 | OUT、STATUS、过流等 |
| 仓门限位/防夹 | 2-4 | 开、关、防夹、备用 |
| UART VSEL/电源开关 | TBD | 取决于每路是否独立控制 |
| 外设电源 Enable | 3-8 | 5V 外设、传感器、电源域 |
| 调试/LED/蜂鸣器 | TBD | 是否需要待定 |

结论：

- H7 建议优先选择高引脚数封装。
- 需要 Ethernet 网口 + USB Type-C + 多 SPI/UART + 仓门驱动时，LQFP144 可能紧张，LQFP176/BGA 需要评估。
- 具体型号和封装必须在原理图前冻结。

## 4. STM32G4 双轮 FOC 资源预算

适用于前轮 G4 和后轮 G4。

### 4.1 每个 G4 控制对象

| 通道 | 电机 | 控制 |
|---|---|---|
| A | 左轮 | 速度/力矩/电流 |
| B | 右轮 | 速度/力矩/电流 |

### 4.2 PWM 资源

| 项目 | 每电机 | 双电机合计 | 要求 |
|---|---:|---:|---|
| 三相互补 PWM | 6 | 12 | 高低侧互补、死区、刹车输入 |
| Driver Enable | 1 | 2 | 可被急停硬件拉低 |
| Driver Fault | 1 | 2 | 可中断/锁存 |

关键点：

- 需要确认所选 G4 是否有足够高级定时器同时支持双电机互补 PWM。
- PWM 硬件 Break 输入应接入过流/急停/Fault。

### 4.3 ADC/采样资源

| 信号 | 每电机 | 双电机合计 | 备注 |
|---|---:|---:|---|
| 相电流采样 | 3 | 6 | 三低边或三相采样 |
| 母线电压 | 可共用 1 | 1 | 每个驱动区至少 1 |
| MOS/板温 | 1-2 | 2-4 | NTC/TBD |
| 电机温度/AUX | 1 | 2 | 取决于轮毂 AUX |

关键点：

- 双 FOC 对 ADC 同步触发要求高。
- 采样放大器输出范围要匹配 ADC。
- 过流比较器最好独立于软件 ADC。

### 4.4 传感器接口

| 信号 | 每电机 | 双电机合计 | 备注 |
|---|---:|---:|---|
| Hall A/B/C | 3 | 6 | 5 V Hall 输入需电平/保护确认 |
| ABI A/B/Z 预留 | 3 | 6 | 可选 |
| SPI 编码器预留 | 4-6 | 8-12 | 可选，需 CS 独立 |

### 4.5 通信/调试

| 功能 | 数量 | 备注 |
|---|---:|---|
| FDCAN | 1 | 内部电机 CAN FD |
| SWD | 1 | 必须有 |
| NRST/BOOT | 1 套 | 量产和调试 |
| UART 调试 | 0-1 | 可选测试点 |

## 5. STM32G4 转向 FOC 资源预算

### 5.1 控制对象

| 通道 | 电机 | 控制 |
|---|---|---|
| A | 前转向 | 输出轴角度位置闭环 |
| B | 后转向 | 输出轴角度位置闭环 |

### 5.2 额外资源

相比轮毂 G4，转向 G4 额外需要：

| 功能 | 数量 | 说明 |
|---|---:|---|
| 输出轴 SPI 编码器 | 2 | 前/后各 1，必须项 |
| 机械限位输入 | 4 | 前左/右限位、后左/右限位，具体机械形式待定 |
| 堵转判断 | 2 | 由电流 + 角度误差 + 时间判断 |
| 绝对零位校准输入/流程 | TBD | 是否需要校准按钮/工装待定 |

### 5.3 转向编码器接口

#### J_STEER_ANGLE_F

| Pin | 信号 | 电平 | 方向 | 保护 |
|---:|---|---|---|---|
| 1 | `3V3_ENC` | 3.3 V | 输出 | 滤波/限流 |
| 2 | `GND` | 0 V | - | - |
| 3 | `SPI_SCK` | 3.3 V | G4 -> ENC | 串阻 + ESD |
| 4 | `SPI_MISO` | 3.3 V | ENC -> G4 | 串阻 + ESD |
| 5 | `SPI_MOSI` | 3.3 V | G4 -> ENC | 可选，串阻 + ESD |
| 6 | `SPI_CS` | 3.3 V | G4 -> ENC | 串阻 + ESD |
| 7 | `DRDY_ERR` | 3.3 V | ENC -> G4 | 可选，上拉/ESD |
| 8 | `SHIELD_GND` | Shield | - | 接地策略待定 |

#### J_STEER_ANGLE_R

同 `J_STEER_ANGLE_F`。

## 6. 外部连接器初版

连接器型号未定，以下仅定义功能和针脚需求。

### 6.1 主电源输入：J_BAT

| Pin | 信号 | 说明 |
|---:|---|---|
| 1 | `VBAT+` | 电池正 |
| 2 | `VBAT-` | 电池负/功率地 |
| 3 | `PE/SHIELD` | 可选，机壳/屏蔽 |

待确认：

- 电流等级。
- 防水等级。
- 是否需要防反插。
- 线径。

### 6.2 电机三相输出

每个电机一组：

- `J_MOT_FL`
- `J_MOT_FR`
- `J_MOT_RL`
- `J_MOT_RR`
- `J_MOT_STF`
- `J_MOT_STR`
- `J_MOT_DOOR`

| Pin | 信号 |
|---:|---|
| 1 | `U` |
| 2 | `V` |
| 3 | `W` |
| 4 | `SHIELD/PE`，可选 |

### 6.3 轮毂 Hall/AUX 接口

每个轮毂一组：

- `J_HALL_FL`
- `J_HALL_FR`
- `J_HALL_RL`
- `J_HALL_RR`

| Pin | 信号 | 说明 |
|---:|---|---|
| 1 | `5V_HALL` | Hall 供电，需限流 |
| 2 | `GND` | 信号地 |
| 3 | `HALL_A` | 需保护/滤波 |
| 4 | `HALL_B` | 需保护/滤波 |
| 5 | `HALL_C` | 需保护/滤波 |
| 6 | `AUX_TEMP_SPEED` | 类型 TBD |
| 7 | `SHIELD_GND` | 可选 |

### 6.4 轮毂编码器预留

每个轮毂建议预留 ABI：

| Pin | 信号 | 说明 |
|---:|---|---|
| 1 | `VCC_ENC` | 3.3 V/5 V 跳线，是否允许 5 V 待确认 |
| 2 | `GND` | - |
| 3 | `ENC_A` | 串阻/ESD/上拉预留 |
| 4 | `ENC_B` | 串阻/ESD/上拉预留 |
| 5 | `ENC_Z` | 串阻/ESD/上拉预留 |
| 6 | `SHIELD_GND` | 可选 |

### 6.5 仓门接口

#### J_DOOR_MOTOR

| Pin | 信号 |
|---:|---|
| 1 | `DOOR_U` |
| 2 | `DOOR_V` |
| 3 | `DOOR_W` |
| 4 | `SHIELD/PE` |

#### J_DOOR_SENSOR

| Pin | 信号 | 说明 |
|---:|---|---|
| 1 | `VCC_DOOR_ENC` | 3.3 V/5 V TBD |
| 2 | `GND` | - |
| 3 | `DOOR_ENC_A` 或 `SPI_SCK` | 由编码器类型决定 |
| 4 | `DOOR_ENC_B` 或 `SPI_MISO` | 由编码器类型决定 |
| 5 | `DOOR_ENC_Z` 或 `SPI_CS` | 由编码器类型决定 |
| 6 | `DOOR_LIMIT_OPEN` | 开门限位 |
| 7 | `DOOR_LIMIT_CLOSE` | 关门限位 |
| 8 | `DOOR_ANTI_PINCH` | 可选 |

### 6.6 电子锁接口：J_LOCK

| Pin | 信号 | 说明 |
|---:|---|---|
| 1 | `V_LOCK+` | 锁供电正 |
| 2 | `LOCK_OUT-` | 低边开关输出 |
| 3 | `LOCK_STATUS` | 锁状态输入 |
| 4 | `GND` | 信号/锁地，接地策略待定 |

### 6.7 CAN 接口

#### 内部 CAN FD

连接 H7 与 3 个 G4，可板内走线，不一定需要外部连接器。

信号：

- `CAN_MOTOR_H`
- `CAN_MOTOR_L`
- `CAN_MOTOR_SHIELD`，可选。

#### 外部 CAN FD：J_CAN_EXT

| Pin | 信号 |
|---:|---|
| 1 | `CAN_EXT_H` |
| 2 | `CAN_EXT_L` |
| 3 | `GND_ISO` 或 `GND` |
| 4 | `SHIELD` |

### 6.8 RS485 接口：J_RS485

| Pin | 信号 |
|---:|---|
| 1 | `RS485_A` |
| 2 | `RS485_B` |
| 3 | `GND` |
| 4 | `SHIELD` |

### 6.9 超声波 UART 接口

共 8 路：

- `J_USONIC_1` 到 `J_USONIC_8`

每路：

| Pin | 信号 | 说明 |
|---:|---|---|
| 1 | `VOUT` | 3.3 V/5 V 跳线 |
| 2 | `GND` | - |
| 3 | `TX_TO_SENSOR` | 板 -> 传感器，经电平转换 |
| 4 | `RX_FROM_SENSOR` | 传感器 -> 板，经电平转换 |
| 5 | `SHIELD_GND` | 可选 |

### 6.10 扩展 UART 接口

共 4 路：

- `J_UART_EXT_1` 到 `J_UART_EXT_4`

每路同超声波 UART。

### 6.11 USB Type-C

接口名：`J_USB_TYPEC`

| 信号 | 说明 |
|---|---|
| `USB_D+` | USB 2.0 差分，低电容 ESD，差分布线 |
| `USB_D-` | USB 2.0 差分，低电容 ESD，差分布线 |
| `CC1` | Type-C 设备侧 CC，下拉 Rd |
| `CC2` | Type-C 设备侧 CC，下拉 Rd |
| `VBUS` | 插入检测为主，不默认给整板供电 |
| `GND` | - |
| `SHIELD` | 机壳/信号地接地策略待定 |

待确认：

- USB 是否只做 FS Device/CDC，还是需要 HS。
- 是否需要从 Type-C 取 5 V 给少量调试电路供电。
- Type-C 外壳是否与金属外壳连接。

### 6.12 Ethernet

接口名：`J_ETH`

必须提供 1 个板载 Ethernet 网口。

默认方向：

- H7 Ethernet MAC + 外置 PHY。
- 优先 RMII，MII 作为备选。
- 优先板载 RJ45；RJ45 可选带磁性器件，或使用外置网络变压器。
- ESD/浪涌保护和屏蔽接地策略必须预留。

待确认：

- PHY 型号。
- RMII/MII。
- RJ45 带磁性器件还是外置变压器。
- RJ45 是否需要带灯。
- 是否需要 PoE，不建议首版加入，除非明确需要。

## 7. 急停与安全信号

### 7.1 急停输入

| 信号 | 方向 | 说明 |
|---|---|---|
| `ESTOP_IN1` | 输入 | 急停链路 1 |
| `ESTOP_IN2` | 输入 | 急停链路 2 |
| `ESTOP_LATCH` | 内部 | 急停锁存 |
| `ESTOP_RESET` | 输入/TBD | 是否独立复位待定 |

### 7.2 硬件关断输出

| 信号 | 目标 |
|---|---|
| `DRV_EN_FRONT` | 前轮驱动区 |
| `DRV_EN_REAR` | 后轮驱动区 |
| `DRV_EN_STEER` | 转向驱动区 |
| `DRV_EN_DOOR` | 仓门驱动区 |
| `CONTACTOR_EN` | 主接触器驱动 |

### 7.3 Fault 回读

| 信号 | 来源 |
|---|---|
| `FAULT_FRONT` | 前轮 G4/Driver |
| `FAULT_REAR` | 后轮 G4/Driver |
| `FAULT_STEER` | 转向 G4/Driver |
| `FAULT_DOOR` | 仓门 Driver |
| `FAULT_POWER` | 电源安全区 |
| `FAULT_LOCK` | 锁过流/状态异常 |

## 8. 后续引脚分配输出格式

确认 MCU 型号后，建议生成以下表：

| MCU | Pin | Signal | Function | Direction | Voltage | Pull | Default State | Safety Critical | Notes |
|---|---|---|---|---|---|---|---|---|---|

强制要求：

- 所有 Enable 默认禁用。
- 所有 Boot/Reset 有明确上拉/下拉。
- 所有外部输入有保护。
- 所有 PWM 输出在复位态不会误开启驱动。
