# 外卖小车底盘控制大板

当前工作区是底盘控制大板的硬件设计资料区。原始需求来自：

- [新建 文本文档.txt](./新建%20文本文档.txt)

## 当前状态

- 已有：系统级方案文字稿。
- 尚无：原理图、PCB、BOM、固件工程、仿真、测试记录。
- 当前目标：先把方案固化成可执行的工程文档，再进入 EasyEDA 原理图设计。

## 文档目录

| 文件 | 用途 |
|---|---|
| [docs/00_project_plan.md](docs/00_project_plan.md) | 项目阶段计划、交付物、风险和通过标准 |
| [docs/01_hardware_design_spec.md](docs/01_hardware_design_spec.md) | 硬件设计规格书初版 |
| [docs/02_interfaces_and_pin_budget.md](docs/02_interfaces_and_pin_budget.md) | 接口、连接器、MCU 外设和引脚资源预算，含必选 Ethernet 网口和 USB Type-C |
| [docs/03_bom_selection.md](docs/03_bom_selection.md) | 器件选型方向、候选器件和待确认条件 |
| [docs/04_open_questions.md](docs/04_open_questions.md) | 进入原理图前必须确认的问题 |
| [docs/05_review_and_bringup_checklist.md](docs/05_review_and_bringup_checklist.md) | 原理图/PCB 审查与样板上电调试清单 |
| [docs/06_decision_log.md](docs/06_decision_log.md) | 已冻结/待冻结的关键工程决策 |
| [docs/07_power_budget.md](docs/07_power_budget.md) | 电源预算、功率计算和待填参数 |
| [docs/08_can_protocol.md](docs/08_can_protocol.md) | H7 与 G4 的 CAN FD 协议骨架 |
| [docs/09_firmware_requirements.md](docs/09_firmware_requirements.md) | H7/G4/仓门/电锁固件需求初版 |
| [docs/10_system_diagrams_and_easyeda_plan.md](docs/10_system_diagrams_and_easyeda_plan.md) | 系统图、PCB 分区草案和 EasyEDA 落图计划 |
| [docs/11_safety_chain_detail.md](docs/11_safety_chain_detail.md) | 急停、安全链路、Fault 锁存和接触器约束 |
| [docs/12_motor_drive_schematic_template.md](docs/12_motor_drive_schematic_template.md) | 双 FOC/转向/仓门驱动原理图模板 |
| [docs/13_connector_harness_matrix.md](docs/13_connector_harness_matrix.md) | 外部连接器、线束、电流等级和屏蔽需求 |
| [docs/14_mcu_resource_matrix.md](docs/14_mcu_resource_matrix.md) | H7/G4 外设资源预算和选型风险 |
| [docs/15_requirements_traceability.md](docs/15_requirements_traceability.md) | 原始需求到设计文档和验证项的追踪表 |
| [docs/16_user_input_form.md](docs/16_user_input_form.md) | 用户需要补充的关键参数填空表 |
| [docs/17_pcb_layout_constraints.md](docs/17_pcb_layout_constraints.md) | PCB 分区、层叠、功率、采样和通信布线约束 |
| [docs/18_schematic_net_naming_and_page_contracts.md](docs/18_schematic_net_naming_and_page_contracts.md) | 原理图网名、页面接口和信号方向规范 |
| [docs/19_bom_template.md](docs/19_bom_template.md) | BOM 字段、关键器件框架和审查规则 |
| [docs/20_risk_register.md](docs/20_risk_register.md) | 风险登记表、RPN 评分和缓解措施 |
| [docs/21_easyeda_execution_plan.md](docs/21_easyeda_execution_plan.md) | EasyEDA 建项、开画顺序和交付物 |
| [docs/22_review_gates.md](docs/22_review_gates.md) | 从需求冻结到样板上电的审查关卡 |

## 工作原则

1. 已明确的需求直接写入规格。
2. 未明确的参数标记为 `TBD`，并进入待确认问题。
3. 高功率、急停、安全、FOC 采样、电源保护相关内容按高风险项处理。
4. 第一版目标是可审查、可打样，不承诺一次板全功能稳定。
5. 原理图开画前，至少要冻结板框、电源输入、电机峰值电流、连接器、铜厚、散热方式和核心 MCU 封装。

## 下一步

建议先回答 [docs/04_open_questions.md](docs/04_open_questions.md) 里的 `P0` 问题。回答后可进入：

1. H7/G4 引脚分配表定稿。
2. 电源树电流预算。
3. 前轮双 FOC 原理图页模板。
4. 后轮/转向/仓门驱动页复用与差异化设计。
