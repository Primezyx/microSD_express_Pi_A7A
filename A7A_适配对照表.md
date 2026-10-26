# microSD_express_Pi 适配 Radxa Cubie A7A —— 逐脚对照表

结论：**不需要改板**。连接器完全一致，引脚定义逐条对上，可以直接用。

---

## 一、连接器对比

| 项目 | 原项目 J2 | Radxa Cubie A7A J3 | 是否一致 |
|---|---|---|---|
| 位数 | 16 | 16（FPC_2X8P） | 是 |
| 触点间距 | 0.5 mm | 0.5 mm | 是 |
| 单排长度 | 15 × 0.5 = 7.5 mm | 8 × 0.5 = 3.5 mm | 否（见下） |
| 安装面 | B.Cu（背面） | — | 转接板背面朝下插 |
| 固定脚 | 无 | SH1 / SH2 接地 | 转接板无固定脚，靠排线保持力 |

原项目 J2 封装取自 `1Library.pretty/PCIe_connector.kicad_mod`，16 个 SMD 焊盘，
x 坐标从 +3.75 到 −3.75，步距 0.5 mm，焊盘 0.35 × 2.6 mm。
这是**单排 16 触点**结构，和树莓派 5 的 PCIe FPC 座是同一套定义。

A7A 原理图（`radxa_cubie_a7a_v1.10_schematic.pdf`，第 16 页
`16_PCIE_40PIN_CONNECT`）标注的是 `FPC_2X8P`，即 2×8。
但**双排底座的 16 个引脚序号与单排 16 触点的序号定义相同**，
所以信号可以逐脚对上。

---

## 二、逐脚对照

| 脚号 | A7A J3 信号 | 原项目 J2 网络 | 对应 Pi 5 定义 | 结论 |
|---|---|---|---|---|
| 1 | 5V1 | +5V | 5V | 一致 |
| 2 | 5V2 | +5V | 5V | 一致 |
| 3 | GND1 | GND | GND | 一致 |
| 4 | CLKP | /PCIE_CLK_P | CLK_P | 一致 |
| 5 | CLKN | /PCIE_CLK_N | CLK_N | 一致 |
| 6 | GND2 | GND | GND | 一致 |
| 7 | RXP | /PCIE_RX_P | RX_P | 一致 |
| 8 | RXN | /PCIE_RX_N | RX_N | 一致 |
| 9 | GND9 | GND | GND | 一致 |
| 10 | TXP | /PCIE_TX_P | TX_P | 一致 |
| 11 | TXN | /PCIE_TX_N | TX_N | 一致 |
| 12 | GND12 | GND | GND | 一致 |
| 13 | POWER_EN | /PCIE_PWR_EN | PWR_EN | 一致 |
| 14 | WAKE | /PCIE_DET_WAKE | WAKE / DET | 一致 |
| 15 | CLKREQ | /PCIE_CLKREQ_N | CLKREQ | 一致 |
| 16 | RESET | /PCIE_RST_B | PERST | 一致 |

**16 个脚全部一致。** 差分对极性没有交叉，供电脚位置相同，四根控制信号位置相同。

---

## 三、供电能力

A7A 侧：5V1 / 5V2 经 FB4（60R 4A 磁珠）取自 VCC5V0_SYS，
去耦 C87 22uF 25V + C88 100nF 50V + C89 22uF 25V。
比 Pi 5 那两个额定 500mA 的 5V 脚宽裕，microSD Express 卡 1.8W 峰值无压力。

---

## 四、阻抗与叠层（沿用原参数，不要改）

- 差分线宽：**0.12192 mm**
- 参考平面：B.Cu 的地平面
- 介质：**50 μm PI**（聚酰亚胺）
- 铜厚：**0.5 oz**
- 走线区域启用净空区（keepout），min_thickness 0.0254 mm

这组参数是按 50 μm PI + 0.5oz 铜算出来的 90Ω 差分阻抗。
**换叠层或换板厂必须重算阻抗**，否则 PCIe 3.0 眼图会崩。

---

## 五、仍需实机验证的两件事

1. **A733 的 PCIe 控制器能否枚举 microSD Express 卡。**
   Radxa 官方文档写明「目前 NVMe SSD 只可以作为拓展存储空间使用，正在适配中」，
   说明 A733 的 PCIe 外设生态还不成熟。协议层面是标准 PCIe，但没有实测先例。

2. **连接器机械保持力。**
   转接板只有 16 个 SMD 焊盘，没有 J3 那样的 SH1 / SH2 接地固定脚。
   FPC 排线插上后的抗拉能力依赖排线自身保持力。

---

## 六、建议的验证顺序

1. 先查 A7A 的 PCIe 相关设备树，确认 `PCIE_PERSTN`、`PCIE_CLKREQ` 是否已启用
2. 用普通 M.2 NVMe 转接板确认 A733 的 PCIe 通道能正常工作
3. 再插 microSD Express 转接板，`lspci` 看能否枚举
4. 能枚举再测速，期望值受 A733 的 PCIe 3.0 x1 实现限制

---

数据来源：
- 原项目 https://github.com/vapetrov/microSD_express_Pi
- A7A 原理图 https://dl.radxa.com/cubie/a7a/docs/hw/radxa_cubie_a7a_v1.10_schematic.pdf
- 树莓派 PCIe 连接器规范 https://datasheets.raspberrypi.com/pcie/pcie-connector-standard.pdf