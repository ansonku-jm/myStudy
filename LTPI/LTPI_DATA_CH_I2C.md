# AST2700 vs AST1700 I2C/SMBus Controller 完整對照

分析來源：
- `AST1700_A1_Datasheet_v10_clean2.pdf` Chapter 18
- `AST2700_A1_Datasheet_v10_clean2_2.pdf` Chapter 68

---

## 一、系統架構概述

| 項目 | AST1700 I2C | AST2700 I2C (Ch68) |
|------|:-----------:|:------------------:|
| Global Register 基址 | `0x30C0F000` | `0x14C0F000` |
| Device 數量 | 16（Device 0~15） | 16（Device 0~15） |
| Device N 基址 | `0x30C0F000 + (N+1)×0x100` | `0x14C0F000 + (N+1)×0x100` |
| I2C Master | 支援，16 buses | 支援，16 buses |
| I2C Slave | 支援，16 buses | 支援，16 buses |
| SMBus | 支援，16 buses | 支援，16 buses |
| HS Mode | 支援（3.4 Mbps） | 支援（3.4 Mbps） |
| DMA Buffer 最大 | 4096 Bytes | 4096 Bytes |
| Pool Buffer 最大 | 32 Bytes | 32 Bytes |

> 兩者 register 結構完全相同，主要差異為 base address。
> AST1700 I2C 位於遠端 IO Expander，透過 LTPI AHB bridge 映射至 AST2700 位址空間（`0x30C0F000`）；AST2700 I2C 為 BMC SoC 本地控制器。

---

## 二、Register 總覽

### 2.1 Address Layout（兩者相同）

| Offset | Size | 描述 |
|--------|------|------|
| `0x0000`~`0x00FF` | 256B | Global Register |
| `0x0100`~`0x01FF` | 256B | Device 0 |
| `0x0200`~`0x02FF` | 256B | Device 1 |
| … | … | … |
| `0x1000`~`0x10FF` | 256B | Device 15 |

### 2.2 Global Cluster Registers（I2CG00~I2CG20，兩者完全相同）

| Offset | 名稱 | Init | 說明 |
|--------|------|------|------|
| `00h` | I2CG00 | 0x0 | Device Master Mode Interrupt Status Register |
| `04h` | I2CG04 | 0x0 | Device Slave Mode Interrupt Status Register |
| `0Ch` | I2CG0C | 0x4 | Global Control Register |
| `10h` | I2CG10 | 0x0 | New Clock Divider Control Register |
| `14h` | I2CG14 | 0x66666666 | I2CG_FIFO_CFG0 |
| `18h` | I2CG18 | 0x66666666 | I2CG_FIFO_CFG1 |
| `1Ch` | I2CG1C | 0x0 | I2CG_MARB_CFG |
| `20h` | I2CG20 | 0x0 | Write lock protection Register for Security |

### 2.3 Device Registers（I2C00~I2C9C，兩者完全相同）

| Offset | 名稱 | Init | 說明 |
|--------|------|------|------|
| `00h` | I2C00 | 0x0 | Master/Slave Function Control Register |
| `04h` | I2C04 | 0x8 | Master/Slave Clock and AC Timing Control Register |
| `08h` | I2C08 | 0x0 | Master/Slave Transmit/Receive Byte Buffer Register |
| `0Ch` | I2C0C | 0x0 | I2CC_BUFCTL |
| `10h` | I2C10 | 0x0 | Master Interrupt Control Register |
| `14h` | I2C14 | 0x0 | Master Interrupt Status Register |
| `18h` | I2C18 | 0x0 | Master Command/Status Register |
| `1Ch` | I2C1C | 0x0 | Master DMA Buffer Length Register |
| `20h` | I2C20 | 0x0 | Slave Interrupt Control Register |
| `24h` | I2C24 | 0x0 | Slave Interrupt Status Register |
| `28h` | I2C28 | 0x0 | Slave Command/Status Register |
| `2Ch` | I2C2C | 0x0 | Slave DMA Buffer Length Register |
| `30h` | I2C30 | 0x0 | Master DMA Mode Tx Buffer Base Address |
| `34h` | I2C34 | 0x0 | Master DMA Mode Rx Buffer Base Address |
| `38h` | I2C38 | 0x0 | Slave DMA Mode Tx Buffer Base Address |
| `3Ch` | I2C3C | 0x0 | Slave DMA Mode Rx Buffer Base Address |
| `40h` | I2C40 | 0x0 | Slave Device Address Register |
| `44h` | I2C44 | 0x0 | Slave Device Address Register (extra) |
| `48h` | I2C48 | 0x0 | Master DMA Length Status Register |
| `4Ch` | I2C4C | 0x0 | Slave DMA Length Status Register |
| `50h` | I2C50 | 0x404 | I2CC_DMA_CFG |
| `54h` | I2C54 | 0x0 | I2CC_DMA_STA |
| `60h` | I2C60 | 0x0 | Master DMA Mode Tx Buffer Base Address[39:32] |
| `64h` | I2C64 | 0x0 | Master DMA Mode Rx Buffer Base Address[39:32] |
| `68h` | I2C68 | 0x0 | Slave DMA Mode Tx Buffer Base Address[39:32] |
| `6Ch` | I2C6C | 0x0 | Slave DMA Mode Rx Buffer Base Address[39:32] |
| `70h` | I2C70 | 0x2fff | MISC configuration for AC timing0 |
| `74h` | I2C74 | 0x0 | MISC configuration for AC timing1 |
| `78h` | I2C78 | 0x0 | Debug information for device |
| `7Ch` | I2C7C | 0x00eff000 | I2CC_HS_ACTIME |
| `80h` | I2C80 | 0x0 | Recorder information for Slave address matching |
| `84h` | I2C84 | 0x0 | Recorder information for Byte transfer |
| `88h` | I2C88 | 0x0 | I2CC_MIRQ_LOG |
| `8Ch` | I2C8C | 0x0 | I2CC_SIRQ_LOG |
| `90h` | I2C90 | 0x0 | I2CC_WLOCK |
| `94h` | I2C94 | 0xffffffff | I2CC_VERSION_CTL |
| `98h` | I2C98 | 0x0 | I2CM_MCMDQ |
| `9Ch` | I2C9C | 0x1020 | I2CC_MISC2 |

---

## 三、Global Cluster Register 詳述（兩者相同）

### 3.1 I2CG00：Device Master Mode Interrupt Status Register

**Offset: 00h，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:16 | RO | Reserved |
| 15:0 | RO | **I2C_MIRQ**：I2C/SMBus Master Interrupt for device[15:0] |

### 3.2 I2CG04：Device Slave Mode Interrupt Status Register

**Offset: 04h，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:16 | RO | Reserved |
| 15:0 | RO | **I2C_SIRQ**：I2C/SMBus Slave Interrupt for device[15:0] |

### 3.3 I2CG0C：Global Control Register

**Offset: 0Ch，Init = 0x00000004**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:12 | RO | Reserved |
| 11:8 | RW | **MTX2RX_DLY**：Master Tx→Rx delay（packet mode）0000=no delay，0001=1 cycle，…，1111=15 cycles |
| 7:5 | RO | Reserved |
| 4 | RW | **SRX_MODE**：0: issue packet done IRQ + pull SCL low；1: expect Repeat Start or Stop |
| 3 | RW | **IRQ_SEPARATE**：0: Master/Slave IRQ merged on I2CG00；1: Master on I2CG00，Slave on I2CG04 |
| 2 | RO | **NEW_REG_MODE** |
| 1 | RW | **CLK_DIV_SEL**：0: Old clock divider mode；1: New clock divider mode |

### 3.4 I2CG10：New Clock Divider Control Register

**Offset: 10h，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:24 | RW | **BASE_DIV4**：Base clock 4 divisor |
| 23:16 | RW | **BASE_DIV3**：Base clock 3 divisor |
| 15:8 | RW | **BASE_DIV2**：Base clock 2 divisor |
| 7:0 | RW | **BASE_DIV1**：Base clock 1 divisor（0x00=/1，0x01=/1.5，0x02=/2，…，0xFF=/128.5） |

### 3.5 I2CG14/18：I2CG_FIFO_CFG0/1

**Offset: 14h / 18h，Init = 0x66666666**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:0 | RW | **FIFO_CFG0 / FIFO_CFG1** |

### 3.6 I2CG1C：I2CG_MARB_CFG

**Offset: 1Ch，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:0 | RW | **XARB_CFG** |

### 3.7 I2CG20：Write Lock Protection Register

**Offset: 20h，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:4 | RO | Reserved |
| 3:0 | RW01 | **GWLOCK** |

---

## 四、Device Register 重點詳述（兩者相同）

### 4.1 I2C00：Master/Slave Function Control Register

**Offset: 00h，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:21 | RO | Reserved |
| 20 | RW | **SLV_ADDR_SAVE_EN**：0: 不儲存 address byte；1: 儲存 |
| 19:18 | RW | **MPKG_RETRY**：00: No retry；01: 1 time；10: 2 times；11: 3 times |
| 17 | RW | **AUTO_RELEASE**：Bus auto-release（SCL low / SDA low / slave inactive timeout） |
| 16 | RW | **AUTO_RECOVERY**：Master auto SDA lock recovery（single master only） |
| 15 | RW | **MULTI_MST_DIS**：0: Enable multi-master；1: Disable（single master only） |
| 14 | RW | **SCL_DRIVE_EN**：SCL direct drive（single master，no clock stretch） |
| 13:10 | RO | Reserved |
| 9 | RW | **SLV_RNGA**：0: normal address；1: range mode（address1~address2） |
| 8 | RW | **SDA_PU**：SDA active drive high for 1T |
| 7 | RW | **SCL_PU**：SCL active drive high for 1T（Master Only） |
| 6 | RW | **HS_MODE**：0: normal；1: HS mode（3.4 Mbps，buffer/DMA only） |
| 5:4 | RW | **DEBOUNCE**：00: 2T；01: 4T；10: 8T；other: DEB_TH×T |
| 3:2 | RO | Reserved |
| 1 | RW | **SLV_EN**：Enable slave function |
| 0 | RW | **MST_EN**：Enable master function |

### 4.2 I2C04：Master/Slave Clock and AC Timing Control Register

**Offset: 04h，Init = 0x00000008**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:30 | RO | Reserved |
| 29:24 | RW | **TTIMEOUT**：reserved |
| 23:20 | RW | **TCKHIGHMIN**：SCL 高電平最短寬度（0=no guarantee，0011=4T，…，1111=16T） |
| 19:16 | RW | **TCKHIGH**：SCL 高電平寬度 |
| 15:12 | RW | **TCKLOW**：SCL 低電平寬度 |
| 11:10 | RW | **THDDAT**：Data hold time |
| 9:8 | RW | **TIMEOUT_BASE**：Timeout period = (timeout_base+1)×(100000/4×PCLK period) |
| 7:0 | RW | **ACTIME_BASE**：AC_baseclock = PCLK / (actime_base+1) |

### 4.3 I2C40/44：Slave Device Address Register

**Offset: 40h，Init = 0x00000000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31 | RW | **SADDR3_EN** |
| 30:24 | RW | **SADDR3**：Slave Device Address3（7-bit） |
| 23 | RW | **SADDR2_EN** |
| 22:16 | RW | **SADDR2** |
| 15 | RW | **SADDR1_EN** |
| 14:8 | RW | **SADDR1** |
| 7 | RW | **SADDR0_EN** |
| 6:0 | RW | **SADDR0** |

**Offset: 44h，Init = 0x00000000**（extra：SADDR4~SADDR7，結構相同）

### 4.4 I2C70：MISC configuration for AC timing0

**Offset: 70h，Init = 0x00002fff**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:20 | RO | Reserved |
| 19:16 | RW | **IO_CFG**：[3] fix sda_o/scl_o=0；[1] use 1T to avoid sda glitch；[0] use 1T to avoid scl glitch |
| 15:12 | RW | **TSUDAT**：Data setup time |
| 11:8 | RW | **TBUF**：Bus free time between Stop and Start |
| 7:4 | RW | **THDSTA**：Hold time after Start |
| 3:0 | RW | **TACST**：Setup time for Stop and repeat Start |

### 4.5 I2C7C：I2CC_HS_ACTIME

**Offset: 7Ch，Init = 0x00eff000**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:0 | RW | **HS_ACTIME**：High Speed AC timing |

### 4.6 I2C94：I2CC_VERSION_CTL

**Offset: 94h，Init = 0xffffffff**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:0 | RW | **FUNC_CFG**：32 個 controller 功能行為旗標（異常偵測、DMA 模式選擇等） |

### 4.7 I2C9C：I2CC_MISC2

**Offset: 9Ch，Init = 0x00001020**

| Bit | R/W | 說明 |
|-----|-----|------|
| 31:16 | RO | Reserved |
| 15:8 | RW | **SP_TH**：避免 poor AC timing 造成誤 Start/Stop |
| 7:0 | RW | **DEB_TH**：DEBOUNCE=3 時，定義最大 glitch 寬度 |

---

## 五、主要差異總結

| 差異點 | AST1700 | AST2700 |
|--------|---------|---------|
| Global Register 基址 | `0x30C0F000` | `0x14C0F000` |
| 晶片角色 | IO Expander（遠端，透過 LTPI AHB bridge） | BMC SoC（本地） |
| Device 數量 | 16（Device 0~15） | 16（Device 0~15） |
| Address Layout（Offset） | **兩者完全相同** | **兩者完全相同** |
| Global Cluster Registers | **兩者完全相同** | **兩者完全相同** |
| Device Registers（I2C00~I2C9C） | **兩者完全相同** | **兩者完全相同** |
| Init 值 | 兩者相同 | 兩者相同 |
