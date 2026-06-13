# 需求追踪表

## 1. 目的

把原始方案中的每条关键需求映射到设计文档、原理图页和验证项目。

状态定义：

- `Captured`：已记录。
- `Specified`：已转成设计规格。
- `Blocked`：缺用户参数，不能继续细化。
- `ReadyForSchematic`：可进入原理图。
- `NeedsReview`：需要专项审查。

## 2. 系统架构需求

| Req ID | 需求 | 状态 | 文档位置 | 验证方式 |
|---|---|---|---|---|
| SYS-001 | 一块大 PCB，内部模块化分区 | Specified | `01_hardware_design_spec.md` | PCB 分区审查 |
| SYS-002 | 上位机只和 STM32H7 通信 | Specified | `01_hardware_design_spec.md` | 原理图/协议审查 |
| SYS-003 | ESP32 只服务 H7 | Specified | `01_hardware_design_spec.md` | 原理图审查 |
| SYS-004 | 不做 7 个电机 MCU | Specified | `06_decision_log.md` | 架构审查 |
| SYS-005 | 不做单 MCU 控全部底盘电机 | Specified | `06_decision_log.md` | 架构审查 |

## 3. MCU 分工需求

| Req ID | 需求 | 状态 | 文档位置 | 验证方式 |
|---|---|---|---|---|
| MCU-001 | H7 负责通信、传感器、调度、安全、仓门、电锁 | Specified | `01_hardware_design_spec.md`, `14_mcu_resource_matrix.md` | PinMap 审查 |
| MCU-002 | 前轮 G4 控制前左+前右 FOC | NeedsReview | `12_motor_drive_schematic_template.md` | G4 资源审查 |
| MCU-003 | 后轮 G4 控制后左+后右 FOC | NeedsReview | `12_motor_drive_schematic_template.md` | G4 资源审查 |
| MCU-004 | 转向 G4 控制前/后转向 FOC | NeedsReview | `12_motor_drive_schematic_template.md` | G4 资源审查 |
| MCU-005 | ESP32-S3 只做 Wi-Fi AP + 蓝牙 | Specified | `01_hardware_design_spec.md` | 原理图审查 |
| MCU-006 | SPI-UART 扩展 8 路超声波 | Captured | `02_interfaces_and_pin_budget.md` | 器件/协议审查 |

## 4. 电机需求

| Req ID | 需求 | 状态 | 文档位置 | 阻塞项 |
|---|---|---|---|---|
| MOT-001 | 4 个 24-48 V 500 W 轮毂 | Captured | `07_power_budget.md` | 峰值电流 |
| MOT-002 | 轮毂三相 U/V/W | Specified | `13_connector_harness_matrix.md` | 连接器 |
| MOT-003 | 轮毂 6 根 Hall 线 | Captured | `13_connector_harness_matrix.md` | AUX 类型 |
| MOT-004 | 支持电子刹车 | Blocked | `08_can_protocol.md` | 刹车策略 |
| MOT-005 | 两个转向无刷位置 FOC | Blocked | `12_motor_drive_schematic_template.md` | 电机/编码器参数 |
| MOT-006 | 转向必须测输出轴 | Specified | `06_decision_log.md` | 编码器型号 |
| MOT-007 | 仓门 BLDC 由 H7 直控 | Blocked | `09_firmware_requirements.md` | 仓门电机参数 |
| MOT-008 | 电子锁由 H7 控制 | Blocked | `01_hardware_design_spec.md` | 锁参数 |

## 5. 通信需求

| Req ID | 需求 | 状态 | 文档位置 | 阻塞项 |
|---|---|---|---|---|
| COM-001 | CAN FD，不用 TJA1050 | Specified | `03_bom_selection.md` | 速率/拓扑 |
| COM-002 | 内部电机 CAN FD | Captured | `08_can_protocol.md` | CAN ID/周期 |
| COM-003 | 外部 CAN FD 建议隔离 | Captured | `03_bom_selection.md` | 是否必须隔离 |
| COM-004 | 8 路 UART 超声波 | Captured | `13_connector_harness_matrix.md` | 模块型号/波特率 |
| COM-005 | 4 路扩展 UART | Captured | `02_interfaces_and_pin_budget.md` | H7 UART 资源 |
| COM-006 | 1 路 RS485 | Captured | `02_interfaces_and_pin_budget.md` | 隔离/速率 |
| COM-007 | ESP32 UART | Specified | `01_hardware_design_spec.md` | 协议 |
| COM-008 | USB Type-C | Specified | `01_hardware_design_spec.md`, `02_interfaces_and_pin_budget.md` | FS/HS、VBUS 供电策略、屏蔽接地 |
| COM-009 | Ethernet 网口 | Specified | `01_hardware_design_spec.md`, `14_mcu_resource_matrix.md` | PHY、RMII/MII、RJ45/磁性器件 |

## 6. 电源需求

| Req ID | 需求 | 状态 | 文档位置 | 阻塞项 |
|---|---|---|---|---|
| PWR-001 | 48 V 电池输入，支持 24-48 V | Captured | `07_power_budget.md` | 最高/最低电压 |
| PWR-002 | 功率器件按 75/100 V 余量 | Captured | `03_bom_selection.md` | 具体电池最高电压 |
| PWR-003 | 48 V -> 24 V | Blocked | `07_power_budget.md` | 24 V 电流 |
| PWR-004 | 48 V -> 12/15 V | Blocked | `07_power_budget.md` | Gate Driver 方案 |
| PWR-005 | 12/15 V -> 5 V | Blocked | `07_power_budget.md` | 5 V 负载 |
| PWR-006 | 5 V -> 3.3 V | Blocked | `07_power_budget.md` | 3.3 V 负载 |
| PWR-007 | V_DOOR 5-24 V | Blocked | `01_hardware_design_spec.md` | 独立/板上供电 |
| PWR-008 | V_LOCK 5-24 V | Blocked | `01_hardware_design_spec.md` | 独立/板上供电 |
| PWR-009 | 总保险/反接/预充/接触器 | Blocked | `11_safety_chain_detail.md` | 总电流/接触器 |

## 7. 安全需求

| Req ID | 需求 | 状态 | 文档位置 | 阻塞项 |
|---|---|---|---|---|
| SAF-001 | 急停不能只进 MCU | Specified | `11_safety_chain_detail.md` | 无 |
| SAF-002 | 急停拉低所有驱动 Enable | Specified | `11_safety_chain_detail.md` | Enable 极性 |
| SAF-003 | 急停控制主接触器断开 | Blocked | `11_safety_chain_detail.md` | 接触器类型 |
| SAF-004 | 所有驱动上电默认不使能 | Specified | `11_safety_chain_detail.md` | PinMap 验证 |
| SAF-005 | 过流/过压/欠压/过温保护 | Blocked | `12_motor_drive_schematic_template.md` | 阈值 |
| SAF-006 | CAN 超时保护 | Blocked | `08_can_protocol.md` | 超时动作 |
| SAF-007 | Fault 锁存 | Blocked | `11_safety_chain_detail.md` | 复位策略 |

## 8. PCB 需求

| Req ID | 需求 | 状态 | 文档位置 | 阻塞项 |
|---|---|---|---|---|
| PCB-001 | 主控区远离功率区 | Captured | `10_system_diagrams_and_easyeda_plan.md` | 板框 |
| PCB-002 | ESP32 天线净空 | Captured | `10_system_diagrams_and_easyeda_plan.md` | 天线/外壳 |
| PCB-003 | 功率区靠板边 | Captured | `10_system_diagrams_and_easyeda_plan.md` | 板框/散热 |
| PCB-004 | 每功率区本地母线电容 | Captured | `05_review_and_bringup_checklist.md` | 电流/PWM |
| PCB-005 | Hall/编码器/UART 不穿大电流区 | Captured | `05_review_and_bringup_checklist.md` | 布局 |
| PCB-006 | 每 MCU 预留 SWD | Specified | `14_mcu_resource_matrix.md` | 连接器 |
| PCB-007 | 每电源轨预留测试点 | Specified | `01_hardware_design_spec.md` | 测试点布局 |

## 9. 当前阻塞原理图的最高优先级

1. 电池最高/最低电压。
2. 轮毂峰值电流。
3. 板框、层数、铜厚、散热。
4. H7/G4 封装接受范围。
5. 转向/仓门/电子锁参数。
6. 急停和接触器策略。
