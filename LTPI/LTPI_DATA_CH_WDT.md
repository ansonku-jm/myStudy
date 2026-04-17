# AST2700 vs AST1700 WDT 完整對照

分析來源：
- `AST1700_A1_Datasheet_v10_clean2.pdf` Chapter 16
- `AST2700_A1_Datasheet_v10_clean2_2.pdf` Chapter 61
- `build/ast2700-default/workspace/sources/linux-aspeed/drivers/watchdog/aspeed_wdt.c`
- `build/ast2700-default/workspace/sources/linux-aspeed/arch/arm64/boot/dts/aspeed/aspeed-g7.dtsi`
- `build/ast2700-default/workspace/sources/linux-aspeed/arch/arm64/boot/dts/aspeed/aspeed-ltpi0.dtsi`

---

## 一、架構總覽

| 項目 | AST1700 WDT | AST2700 WDT |
|------|:-----------:|:-----------:|
| Instance 數量 | 9（WDT0~8，每 instance +0x80） | 9（WDT0~8，每 instance +0x80） |
| DTS compatible | `aspeed,ast2700-wdt` | `aspeed,ast2700-wdt` |
| Reset Mask 數量 | 5 | 5 |
| 晶片角色 | IO Expander（透過 LTPI AHB bridge 存取） | BMC SoC（本地） |

> 兩者 compatible 相同，使用同一 kernel driver（`aspeed_wdt.c`）。Register 結構完全相同，主要差異為 Base Address。

---

## 二、Base Address 對照

| Instance | AST1700（LTPI mapped） | AST2700（本地） |
|----------|:---------------------:|:---------------:|
| WDT0 | `0x30C3_7000` | `0x14C3_7000` |
| WDT1 | `0x30C3_7080` | `0x14C3_7080` |
| WDT2 | `0x30C3_7100` | `0x14C3_7100` |
| WDT3 | `0x30C3_7180` | `0x14C3_7180` |
| WDT4 | `0x30C3_7200` | `0x14C3_7200` |
| WDT5 | `0x30C3_7280` | `0x14C3_7280` |
| WDT6 | `0x30C3_7300` | `0x14C3_7300` |
| WDT7 | `0x30C3_7380` | `0x14C3_7380` |
| WDT8 | `0x30C3_7400` | `0x14C3_7400` |

---

## 三、Register Offset 對照

| Offset | 功能 | AST1700 | AST2700 |
|--------|------|:-------:|:-------:|
| `00h` | WDT Status（counter value，RO） | ✓ | ✓（同） |
| `04h` | WDT Reload Value | ✓ | ✓（同） |
| `08h` | WDT Restart（write 0x4755） | ✓ | ✓（同） |
| `0Ch` | WDT Control Register | ✓ | ✓（同） |
| `10h` | WDT Timeout Status Register | ✓ | ✓（同） |
| `14h` | WDT Clear Timeout Status | ✓ | ✓（同） |
| `18h` | WDT Reset Width（外部 pulse width / 極性 / drive mode） | ✓ | ✓（同） |
| `1Ch` | Reset Mask Register #1 | ✓ | ✓（同） |
| `20h` | Reset Mask Register #2 | ✓ | ✓（同） |
| `24h` | Reset Mask Register #3（I3C bus0~15 等） | ✓ | ✓（同） |
| `28h` | Reset Mask Register #4（VGALink / LTPI / IOMCU / UART 等） | ✓ | ✓（同） |
| `2Ch` | Reset Mask Register #5（I2C bus0~15） | ✓ | ✓（同） |
| `30h` | Software Mode Reset Control Register | ✓ | ✓（同） |
| `34h` | Software Mode Reset Mask Register #1 | ✓ | ✓（同） |
| `38h` | Software Mode Reset Mask Register #2 | ✓ | ✓（同） |
| `3Ch` | Software Mode Reset Mask Register #3 | ✓ | ✓（同） |
| `40h` | Software Mode Reset Mask Register #4 | ✓ | ✓（同） |
| `44h` | Software Mode Reset Mask Register #5 | ✓ | ✓（同） |
| `48h` | Function Disable Control Register（W1S，sticky 直到 SRST#） | ✓ | ✓（同） |
| `4Ch` | Scratch Register | ✓ | ✓（同） |
| `50h~60h` | Reset Mask Write Protection Register #1~#5 | ✓ | ✓（同） |
| `64h~74h` | Software Mode Reset Mask Write Protection Reg #1~#5 | ✓ | ✓（同） |
| `7Ch` | WDT Write Protection Register | ✓ | ✓（同） |

---

## 四、WDT Control Register（offset 0Ch）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:10 | 預觸發中斷計時（interrupt mode 用） | ✓ | ✓（同） |
| 9:8 | Reserved | ✓ | ✓ |
| 7 | Boot from secondary（1: 從 secondary boot） | ✓ | ✓（同） |
| 6:5 | Reset mode（00=SOC / 01=Full / 10=ARM CPU） | ✓ | ✓（同） |
| 4 | RST_SOC | ✓ | ✓（同） |
| 3 | WDT_EXT（enable 外部 pulse 輸出） | ✓ | ✓（同） |
| 2 | WDT_INTR（enable interrupt 輸出） | ✓ | ✓（同） |
| 1 | RESET_SYSTEM（1: enable watchdog reset） | ✓ | ✓（同） |
| 0 | ENABLE | ✓ | ✓（同） |

---

## 五、Software Mode Reset Control Register（offset 30h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:8 | Reserved | ✓ | ✓ |
| 7:4 | Watchdog event counter（W0xDEADDEAD 清除） | ✓ | ✓（同） |
| 3:1 | Reserved | ✓ | ✓ |
| 0 | Enable trigger software mode reset（W0xAEEDF123 觸發） | ✓ | ✓（同） |

---

## 六、Function Disable Control Register（offset 48h）

| Bit | R/W | 功能 | AST1700 | AST2700 |
|-----|-----|------|:-------:|:-------:|
| 5 | RW1S | Disable watchdog Software mode reset | ✓ | ✓（同） |
| 4 | RW1S | Disable watchdog interrupt generation | ✓ | ✓（同） |
| 3 | RW1S | Disable watchdog CPU/FMC reset mode | ✓ | ✓（同） |
| 2 | RW1S | Disable watchdog Full reset mode | ✓ | ✓（同） |
| 1 | RW1S | Disable watchdog SOC reset mode | ✓ | ✓（同） |
| 0 | RW1S | Disable full watchdog function | ✓ | ✓（同） |

> Note：W1S，寫入後 sticky，直到下一次 SRST# 才能解除。

---

## 七、Scratch Register（offset 4Ch）

| Bit | R/W | 功能 | AST1700 | AST2700 |
|-----|-----|------|:-------:|:-------:|
| 31:24 | WO | Write 0xEA 清除 WDT4C[7:0] | ✓ | ✓（同） |
| 7:0 | RW | Scratch Register | ✓ | ✓（同） |

---

## 八、主要差異總結

| 差異點 | AST1700 | AST2700 |
|--------|---------|---------|
| Base Address | `0x30C3_7000`（LTPI mapped，WDT0） | `0x14C3_7000`（本地，WDT0） |
| 晶片角色 | IO Expander（透過 LTPI AHB bridge 存取） | BMC SoC（本地） |
| Register 結構（00h~7Ch） | 兩者完全相同 | 兩者完全相同 |
