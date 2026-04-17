# JTAG Master Controller: AST2700 vs AST1700 完整比較

## 一、系統架構概述

| 項目 | AST2700 (SCM) | AST1700 (HPM) |
| :--- | :--- | :--- |
| 角色 | 主控端，具備 CPU Reset 控制 | 主控端，用於 IO 擴展或邊界掃描 |
| JTAG 實例數 | **2 個（JTAGM 1 / JTAGM 2）** | **1 個（JTAGM）** |
| JTAGM 1 基址 | **`0x12C2_0000`** | — |
| JTAGM 2 基址 | **`0x14C0_9000`** | — |
| 內部 AHB 基址 | — | **`0x30C0_9000`** |
| SCM 側映射基址 | — | **`0x30C0_9000`**（透過 LTPI AHB bridge） |
| DTS Compatible | `aspeed,ast2700-jtag` | `aspeed,ast2700-jtag`（共用同一 driver） |
| LTPI 隧道支援 | 支援（JTAG Over LTPI） | 支援（作為隧道終端） |

> Register 結構兩者**完全相同**，主要差異為基址與實例數量。
> AST1700 JTAG 透過 LTPI AHB bridge 映射至 AST2700 位址空間（`0x30C0_9000`），使用相同的 kernel driver。

---

## 二、Register 總覽

| Offset | 名稱 | Init | 說明 |
| :--- | :--- | :--- | :--- |
| **00h / 04h** | JTAG00 / JTAG04 | 0x0 | Data Port Register（HW Mode 1 資料/指令） |
| **08h** | JTAG08 | 0x0 | Engine Control（HW Mode 1） |
| **0Ch** | JTAG0C | 0x0 | Interrupt Status and Enable |
| **10h** | JTAG10 | 0x0 | Software Mode and Status |
| **14h** | JTAG14 | 0x0007 | TCK Control |
| **18h** | JTAG18 | 0x8000_0000 | Engine Control 1（TRSTn） |
| **20h / 24h** | JTAG20 / JTAG24 | 0x0 | Shift In/Out Data Register（HW Mode 2） |
| **28h** | JTAG28 | 0x0 | Padding Control 0 |
| **2Ch** | JTAG2C | 0x0 | Padding Control 1 |
| **30h** | JTAG30 | 0x0 | Shift Control |
| **34h** | JTAG34 | 0x8007 | Global Control（HW Mode 2） |
| **38h** | JTAG38 | 0x0 | Interrupt Control（HW Mode 2） |
| **3Ch** | JTAG3C | 0x0 | Status |

---

## 三、JTAG00 / JTAG04：Data Port Register（HW Mode 1）

**Offset: 00h / 04h，Init = 0**

JTAG00 用於 Data 傳輸；JTAG04 用於 Instruction 傳輸。

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:0 | RW | **Data Port Register**<br>Write: 送出資料值；Read: 接收資料值 |

---

## 四、JTAG08：Engine Control（HW Mode 1）

**Offset: 08h，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31 | RW | **Engine Enable** |
| 30 | RW | **Engine Output Enable**<br>1: output enable；0: output disable |
| 29 | RW | **Force Reset State by TMS**<br>強制 controller 與 slave 進入 Reset state |
| 28:22 | RO | Reserved |
| 21 | RW | **Reset Internal FIFO**<br>Write 1 to reset（自動清除） |
| 20 | RO | **Internal FIFO Mode**<br>0: CPU mode；1: Controller mode |
| 19:18 | RO | Reserved |
| 17:8 | RW | **Length in One Transmission**<br>0: forbidden；1: 1 bit；512: 512 bits；513 以上不支援 |
| 7 | RO | Reserved |
| 6 | RW | **MSB First**<br>1: MSB first；0: LSB first |
| 5 | RW | **Terminating Transmission**<br>1: terminate；0: normal operation |
| 4 | RW | **Last Transmission**<br>1: last transmission；0: more data waited |
| 3:2 | RO | Reserved |
| 1 | RW | **Enable Transmission of Instruction** |
| 0 | RW | **Enable Transmission of Data** |

---

## 五、JTAG0C：Interrupt Status and Enable

**Offset: 0Ch，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:20 | RO | Reserved |
| 19 | RW | **Instruction Transmission Pause 狀態（W1C）**<br>Read 0: No interrupt；Read 1: interrupt pending；Write 1: Clear |
| 18 | RW | **Instruction Transmission Completed 狀態（W1C）** |
| 17 | RW | **Data Transmission Pause 狀態（W1C）** |
| 16 | RW | **Data Transmission Completed 狀態（W1C）** |
| 15:4 | RO | Reserved |
| 3 | RW | **Enable: Instruction Transmission Pause** |
| 2 | RW | **Enable: Instruction Transmission Completed** |
| 1 | RW | **Enable: Data Transmission Pause** |
| 0 | RW | **Enable: Data Transmission Completed** |

---

## 六、JTAG10：Software Mode and Status

**Offset: 10h，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:20 | RO | Reserved |
| 19 | RW | **Software Mode Enable** |
| 18 | RW | **Software TCK**<br>Software mode 時直接控制 TCK |
| 17 | RW | **Software TMS**<br>Software mode 時直接控制 TMS |
| 16 | RW | **Software TDI / TDO**<br>Write: 直接連接 TDI；Read: 直接讀取 TDO |
| 15:3 | RO | Reserved |
| 2 | RO | **Instruction Transmission Pause（狀態）** |
| 1 | RO | **Data Transmission Pause（狀態）** |
| 0 | RO | **Engine Idle（狀態）** |

---

## 七、JTAG14：TCK Control

**Offset: 14h，Init = 0x0000_0007**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31 | RW | **TCK Inverse**<br>1: Inverse TCK；0: Not inverse |
| 30:11 | RO | Reserved |
| 10:0 | RW | **TCK Divisor**<br>TCK period = Period of HCLK × (JTAG14[10:0] + 1) |

---

## 八、JTAG18：Engine Control 1

**Offset: 18h，Init = 0x8000_0000**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31 | RW | **Control of TRSTn**<br>1: TRSTn is high；0: TRSTn is low |
| 30:0 | RO | Reserved |

---

## 九、JTAG20 / JTAG24：Shift In/Out Data Register（HW Mode 2）

**Offset: 20h / 24h，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:0 | RW | **Shift Data [31:00]**<br>Write: data to shift out；Read: data from shift in |

---

## 十、JTAG28 / JTAG2C：Padding Control 0 / 1

**Offset: 28h / 2Ch，Init = 0**

JTAG28 對應 Padding 設定 0；JTAG2C 對應 Padding 設定 1（由 JTAG30[9] 選擇）。

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:25 | RO | Reserved |
| 24 | RW | **Padding Data** |
| 23:21 | RO | Reserved |
| 20:12 | RW | **Post Padding Number**<br>0: No post padding；1~511: post padding bit 數 |
| 11:9 | RO | Reserved |
| 8:0 | RW | **Pre Padding Number**<br>0: No pre padding；1~511: pre padding bit 數 |

---

## 十一、JTAG30：Shift Control

**Offset: 30h，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31 | RW | **Enable Free Run TCK**<br>1: TCK is free running；0: TCK toggles only when necessary |
| 30 | RW | **Enable Static Shift** |
| 29:16 | RW | **TMS Value** |
| 15:13 | RW | **Post TMS Shift Number**<br>0: no post TMS shift；1~7: shift cycles |
| 12:10 | RW | **Pre TMS Shift Number**<br>0: no pre TMS shift；1~7: shift cycles |
| 9 | RW | **Padding Selection**<br>0: use JTAG28；1: use JTAG2C |
| 8 | RW | **End of Shift** |
| 7 | RW | **Start of Shift** |
| 6:0 | RW | **Lower Data Shift Number**<br>0: no data shift；1~512: data shift bit 數 |

---

## 十二、JTAG34：Global Control（HW Mode 2）

**Offset: 34h，Init = 0x8007**

HW Mode 2 的總控暫存器，兼含 TRSTn 控制與 TCK divisor。

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31 | RW | **Engine and Mode Enable** |
| 30 | RW | **Engine Output Enable**（僅 JTAG34[31]=1 時有效）<br>1: output enable；0: output disable |
| 29 | RW | **Force Reset State by TMS**（僅 JTAG34[31]=1 時有效） |
| 28:26 | RO | Reserved |
| 25 | RW | **Reset Internal FIFO**（Write 1 to reset，auto clear） |
| 24 | RO | **Internal FIFO Mode**<br>0: CPU mode；1: Controller mode |
| 23 | RO | Reserved |
| 22:20 | RW | **Upper Data Shift Number**<br>與 Lower Data Shift Number（JTAG30[6:0]）組合為實際移位總數；0~512 |
| 19:17 | RO | Reserved |
| 16 | RW | **Static Shift Value**<br>當 JTAG30[30]=1 時移出此 bit |
| 15 | RW | **TRST Value** |
| 14:13 | RO | Reserved |
| 12 | RO | **TCK State** |
| 11:0 | RW | **Clock Divisor**<br>TCK period = Period of HCLK × (JTAG34[10:0] + 1) |

---

## 十三、JTAG38：Interrupt Control（HW Mode 2）

**Offset: 38h，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:17 | RO | Reserved |
| 16 | RW | **Shift Complete Interrupt Enable** |
| 15:1 | RO | Reserved |
| 0 | RW | **Shift Complete Interrupt Status（W1C）**<br>Read 0: No interrupt；Read 1: interrupt pending；Write 1: Clear |

---

## 十四、JTAG3C：Status

**Offset: 3Ch，Init = 0**

| Bit | R/W | 說明 |
| :--- | :--- | :--- |
| 31:1 | RO | Reserved |
| 0 | RO | **Engine Idle** |

---

## 十五、主要差異總結

| 差異點 | AST2700 | AST1700 |
| :--- | :--- | :--- |
| 角色 | SCM（系統主控） | HPM（IO 擴展，遠端） |
| JTAG 實例數 | 2（JTAGM 1 / JTAGM 2） | 1 |
| JTAGM 1 基址 | `0x12C2_0000` | — |
| JTAGM 2 基址 | `0x14C0_9000` | — |
| 內部 AHB 基址 | — | `0x30C0_9000` |
| SCM 側映射基址 | — | `0x30C0_9000`（LTPI AHB bridge） |
| DTS Compatible | `aspeed,ast2700-jtag` | `aspeed,ast2700-jtag`（共用） |
| Register 結構（全部 offset） | **兩者完全相同** | **兩者完全相同** |
| Init 值 | 兩者相同 | 兩者相同 |
| 操作模式 | Software / HW Mode 1 / HW Mode 2 | Software / HW Mode 1 / HW Mode 2 |
