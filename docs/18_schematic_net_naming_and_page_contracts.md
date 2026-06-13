# 原理图网名与页面接口规范

## 1. 目的

统一网名、页面接口、信号方向和默认状态，避免多页原理图中出现隐性错误。

## 2. 页面命名

| 页名 | 前缀 | 说明 |
|---|---|---|
| `System_Block` | `SYS` | 框图和说明 |
| `Power_Input_Safety` | `PWR` | 电池、保险、反接、预充、接触器 |
| `Power_Rails` | `PWR` | DCDC/LDO |
| `STM32H7_Main` | `H7` | H7 最小系统 |
| `H7_Comm` | `COM` | USB Type-C/ETH/CAN/RS485 |
| `ESP32_S3` | `ESP` | 无线 |
| `UART_Sensor_Expansion` | `UART` | 超声波和扩展 UART |
| `G4_Front_Dual_FOC` | `G4F` | 前轮驱动 |
| `G4_Rear_Dual_FOC` | `G4R` | 后轮驱动 |
| `G4_Steer_Dual_FOC` | `G4S` | 转向驱动 |
| `Door_Motor_Driver` | `DOOR` | 仓门 |
| `Lock_Driver` | `LOCK` | 电子锁 |
| `Safety_Chain` | `SAFE` | 急停和硬件安全 |
| `Connectors_Testpoints` | `CONN` | 连接器和测试点 |

## 3. 电源网名

| 网名 | 说明 |
|---|---|
| `VBAT` | 电池输入正 |
| `PGND_BAT` | 电池/功率地 |
| `VMOT` | 电机主母线 |
| `24V_SYS` | 系统 24 V |
| `15V_DRV` | Gate Driver 15 V |
| `12V_DRV` | 若最终选 12 V Gate Driver |
| `5V_SYS` | 系统 5 V |
| `5V_HALL` | Hall 供电，建议限流后分支 |
| `3V3_SYS` | 系统 3.3 V |
| `3V3_A` | 模拟低噪声 3.3 V |
| `3V3_ENC` | 编码器 3.3 V |
| `V_DOOR` | 仓门电源 |
| `V_LOCK` | 电子锁电源 |

规则：

- `PGND_BAT`、`PGND_MOTOR`、`GND_SIG` 是否分网由 PCB 策略决定，不能随意短接。
- 若分网，必须有明确单点/磁珠/0R/采样参考策略。
- 不允许多个相似名字代表同一电源，例如 `3.3V`、`VCC3V3`、`3V3` 混用。

## 4. 安全信号命名

| 信号 | 方向 | 默认态 | 说明 |
|---|---|---|---|
| `ESTOP_IN1` | 外部 -> 板 | TBD | 急停通道 1 |
| `ESTOP_IN2` | 外部 -> 板 | TBD | 急停通道 2 |
| `ESTOP_LATCH` | 安全链内部 | Active = 急停 | 锁存状态 |
| `SAFE_DRV_EN_GLOBAL` | 安全链 -> 驱动 | Disabled | 总驱动许可 |
| `DRV_EN_FRONT` | 安全链/H7 -> 前轮 | Disabled | 前轮使能 |
| `DRV_EN_REAR` | 安全链/H7 -> 后轮 | Disabled | 后轮使能 |
| `DRV_EN_STEER` | 安全链/H7 -> 转向 | Disabled | 转向使能 |
| `DRV_EN_DOOR` | 安全链/H7 -> 仓门 | Disabled | 仓门使能 |
| `CONTACTOR_EN` | 安全链/H7 -> 接触器 | Off | 接触器控制 |

规则：

- 所有 Enable 信号必须有复位默认态说明。
- 如果低有效，网名必须带 `_N` 或 `n` 前缀，例如 `DRV_EN_N`，不能含糊。

## 5. 电机通道命名

### 5.1 轮毂

| 通道 | 前缀 |
|---|---|
| 前左轮 | `FL` |
| 前右轮 | `FR` |
| 后左轮 | `RL` |
| 后右轮 | `RR` |

信号模板：

- `{CH}_U`
- `{CH}_V`
- `{CH}_W`
- `{CH}_PWM_UH`
- `{CH}_PWM_UL`
- `{CH}_PWM_VH`
- `{CH}_PWM_VL`
- `{CH}_PWM_WH`
- `{CH}_PWM_WL`
- `{CH}_IU`
- `{CH}_IV`
- `{CH}_IW`
- `{CH}_HALL_A`
- `{CH}_HALL_B`
- `{CH}_HALL_C`
- `{CH}_AUX`
- `{CH}_DRV_FAULT`
- `{CH}_DRV_EN`

### 5.2 转向

| 通道 | 前缀 |
|---|---|
| 前转向 | `STF` |
| 后转向 | `STR` |

信号模板：

- `{CH}_U/V/W`
- `{CH}_PWM_*`
- `{CH}_I*`
- `{CH}_ANGLE_SPI_SCK`
- `{CH}_ANGLE_SPI_MISO`
- `{CH}_ANGLE_SPI_MOSI`
- `{CH}_ANGLE_SPI_CS`
- `{CH}_ANGLE_DRDY_ERR`
- `{CH}_LIMIT_L`
- `{CH}_LIMIT_R`
- `{CH}_DRV_FAULT`

## 6. 页面接口契约

### 6.1 H7 Main 页面输出/输入

必须清楚连接到其他页面：

| 信号 | 页面 |
|---|---|
| `CAN_MOTOR_TX/RX` | H7_Comm / G4 pages |
| `CAN_EXT_TX/RX` | H7_Comm |
| `ESP_UART_TX/RX` | ESP32_S3 |
| `RS485_TX/RX/DE` | H7_Comm |
| `SPI_UART_*` | UART_Sensor_Expansion |
| `IMU_SPI_*` | H7_Main or Sensor |
| `DOOR_PWM_*` | Door_Motor_Driver |
| `LOCK_CTRL` | Lock_Driver |
| `ESTOP_STATUS` | Safety_Chain |
| `CONTACTOR_CMD` | Safety_Chain |

### 6.2 G4 页面接口

每个 G4 页面应有：

- `CAN_MOTOR_H/L`。
- `SAFE_DRV_EN_GLOBAL` 或分区 Enable。
- `FAULT_TO_H7`。
- `VMOT`。
- `15V_DRV/12V_DRV`。
- `5V_HALL`。
- `3V3_SYS`。
- SWD。

## 7. 信号方向标注

原理图符号/页端口必须标方向：

- Input。
- Output。
- Bidirectional。
- Power。
- Passive。

尤其注意：

- Fault 是 Driver/G4 输出到 H7/安全链。
- Enable 是安全链/H7 输出到 Driver。
- Hall 是电机输出到 G4。
- SPI MISO 是外设输出到 MCU。
- UART TX/RX 命名必须说明参考对象。

## 8. UART 命名规则

为避免 TX/RX 反接，外部 UART 使用角色命名：

- `UARTx_TX_TO_SENSOR`
- `UARTx_RX_FROM_SENSOR`

不要只写：

- `UART_TX`
- `UART_RX`

内部 MCU 管脚可另标：

- `H7_UARTx_TX`
- `H7_UARTx_RX`

## 9. CAN 命名规则

逻辑侧：

- `H7_FDCAN1_TX`
- `H7_FDCAN1_RX`
- `G4F_FDCAN_TX`
- `G4F_FDCAN_RX`

总线侧：

- `CAN_MOTOR_H`
- `CAN_MOTOR_L`
- `CAN_EXT_H`
- `CAN_EXT_L`

不要把逻辑 TX/RX 和总线 H/L 混在一个命名层级里。

## 10. USB Type-C 命名规则

接口名：

- `J_USB_TYPEC`

信号名：

- `USB_DP`
- `USB_DM`
- `USB_CC1`
- `USB_CC2`
- `USB_VBUS_DET`
- `USB_SHIELD`

规则：

- `CC1/CC2` 不允许省略，必须有设备侧 Rd 或 Type-C 控制/保护方案。
- `VBUS` 默认作为检测输入，不直接命名为系统 `5V_SYS`。
- ESD 器件应靠近连接器，测试点不应破坏差分连续性。

## 11. Ethernet 命名规则

逻辑/MAC 侧按 H7 外设命名，例如：

- `H7_ETH_REF_CLK`
- `H7_ETH_MDC`
- `H7_ETH_MDIO`
- `H7_ETH_TXD0/TXD1`
- `H7_ETH_RXD0/RXD1`
- `H7_ETH_CRS_DV`
- `H7_ETH_TX_EN`

PHY 到 RJ45/磁性器件侧按差分对命名：

- `ETH_TX_P/N`
- `ETH_RX_P/N`
- `ETH_LED_LINK`
- `ETH_LED_ACT`
- `ETH_SHIELD`

规则：

- MAC 侧 RMII/MII 信号和网口侧差分对不能混名。
- RJ45 屏蔽接地策略必须在原理图备注中说明。

## 12. 测试点命名

规则：

- 电源：`TP_3V3_SYS`
- 信号：`TP_FL_IU`
- Fault：`TP_FAULT_FRONT`
- PWM：`TP_FL_PWM_UH`

测试点不应改变网名含义。

## 13. 原理图审查规则

每页完成后检查：

1. 电源输入/输出是否有网名。
2. 所有页端口方向是否正确。
3. 所有外部输入是否有保护。
4. 所有 Enable 是否默认禁用。
5. 所有 Fault 是否可观测。
6. 所有 MCU 调试脚是否可达。
7. 是否有未命名网络。
8. 是否有 `TBD` 直接进入生产关键电路。
