# 智能花园浇水系统

只在白天、并且土壤干的时候才浇水的小型花园浇水控制板。主控是立创 STM32F103C8T6 开发板。水泵用 5V 微型潜水泵（S8050 驱动），电磁阀目前用 LED 模拟。

状态：硬件设计完成（原理图、PCB、DRC 通过，已生成打板数据）；STM32 程序还没有写。水泵已改为真实 5V 微型潜水泵驱动（见下）。

## 功能

- 光照传感器 BH1750 判断白天或夜里，夜里不浇水。
- 电容式土壤湿度传感器（PA0）判断土壤干湿，水位开关（PA7）判断水箱是否有水。
- 三个条件都满足时，PA4 输出高电平，经 S8050 驱动 5V 微型潜水泵（同时点亮指示 LED）；PA5 点亮电磁阀 LED（电磁阀仍用 LED 模拟）。
- SHT30 测量空气温湿度，0.91 寸 OLED 显示读数和状态。
- 四个按键用来设置阈值：MENU（PB12）切换设置项，UP（PB13）加，DOWN（PB14）减，OK（PB15）确认。可设置土壤干阈值、土壤湿阈值、光照阈值、浇水时间。
- 有源蜂鸣器（PA6）用于提示和报警。

## 引脚分配

| 引脚 | 网络 | 连接 |
|---|---|---|
| PB6 | I2C_SCL | BH1750、SHT30、OLED 插座，4.7k 上拉到 3V3 |
| PB7 | I2C_SDA | 同上 |
| PA0 | SOIL_ADC | 土壤传感器 H1 第 2 脚 |
| PA7 | WATER_LEVEL | 水位开关 U4 第 1 脚（第 2 脚接 GND） |
| PB12 / PB13 / PB14 / PB15 | KEY_MENU / UP / DOWN / OK | SW2 / SW3 / SW4 / SW5，另一脚接 GND |
| PA4 | PUMP_LED | 330Ω（R1）到指示 LED（U5）；同时 1kΩ（R2）到 S8050（Q1）基极，Q1 集电极接水泵（U2 第 2 脚），U2 第 1 脚接 +5V，SS14（D1）反向并在水泵两端续流 |
| PA5 | VALVE_LED | 330Ω（R4）到电磁阀 LED（U6） |
| PA6 | BUZZER | 1k（R3）到 S8050 基极，蜂鸣器接 5V |

供电：USB 5V。传感器、土壤模块和上拉电阻用 3V3，OLED 模块、蜂鸣器和水泵用 5V。

水泵：微型直流潜水泵（标称 3V，USB 供电款，约 100~250mA），接板上 U2 接线端子，红线接第 1 脚（5V），黑线接第 2 脚。5V 比标称 3V 高，请核对商品额定电压范围（常见为 3~6V），不要空转。

I2C 地址：BH1750（ADDR 接地）0x23，SHT30（ADDR 接地）0x44。OLED 模块一般 0x3C。

OLED 插座（U3）引脚顺序：1=GND，2=+5V，3=SCL，4=SDA。模块到货后请核对自己那块的丝印顺序。

## 图

| 文件 | 内容 |
|---|---|
| [`docs/block-diagram.svg`](docs/block-diagram.svg) | 原理框图 |
| [`docs/wiring-diagram.svg`](docs/wiring-diagram.svg) | 按引脚逐根对应的接线图 |
| [`docs/garden-watering-schematic.pdf`](docs/garden-watering-schematic.pdf) | 嘉立创 EDA 导出的原理图 |
| [`docs/garden-watering-pcb.pdf`](docs/garden-watering-pcb.pdf) | 嘉立创 EDA 导出的 PCB（位号、装配图、钻孔图） |

## PCB

- 两层板，81.28 × 81.28 mm。
- 顶层、底层各铺一块 GND 覆铜，DRC 无错误。
- 四个按键 SW2–SW5 排在同一条水平线上。
- 板子左上角丝印“关于智 高浩铭”，右下角丝印“基于STM32的智能花园自动浇水系统”（黑体，已转成矢量）。
- 开发板 U1 插在板上，传感器 U8、U9 贴片焊在板上，OLED 通过 4 针排针 U3 插接。

## 文件

| 路径 | 说明 |
|---|---|
| [`garden-watering-system.epro2`](garden-watering-system.epro2) | 嘉立创 EDA 专业版工程文件，可直接导入 |
| [`fabrication/Gerber_garden-watering_2026-11-23.zip`](fabrication/Gerber_garden-watering_2026-11-23.zip) | **打板用 Gerber（含覆铜），上传嘉立创下单** |
| [`fabrication/BOM_eda.tsv`](fabrication/BOM_eda.tsv) | EDA 导出的 BOM（制表符分隔，原设计元件编号） |
| [`fabrication/netlist.tel`](fabrication/netlist.tel) | 网表（Allegro 格式） |
| [`物资清单.md`](物资清单.md) | 立创商城采购清单，含替换料和价格 |

## 已知问题和待办

- STM32 程序还没有写。
- 原理图里留有几根早期 DHT11 和旧排针的游离线，只产生 DRC 警告，不影响 PCB。
- OLED 模块换成了 0.91 寸（128×32），引脚顺序到货后要核对。
- 电容式土壤湿度传感器模块在立创商城没有，需要另外购买。
- 水泵已有驱动电路；电磁阀目前仍是 LED 模拟，换成真实电磁阀需要另外的驱动（继电器或 MOSFET）和续流二极管。
