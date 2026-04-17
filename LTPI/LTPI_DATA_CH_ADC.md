# AST2700 vs AST1700 ADC 完整對照

分析來源：
- `AST1700_A1_Datasheet_v10_clean2.pdf` Chapter 12
- `AST2700_A1_Datasheet_v10_clean2_2.pdf` Chapter 38

---

## 一、架構總覽

| 項目 | AST1700 ADC | AST2700 ADC (Ch38) |
|------|:-----------:|:------------------:|
| Base Address | `0x30C0_0000` | `0x14C40000` |
| Engine 數量 | 2（Engine 0 / Engine 1） | 2（Engine 0 / Engine 1） |
| 每 Engine channel 數 | 8 | 8 |
| 總 channel 數 | 16（ch0~ch15） | 16（ch0~ch15） |
| 解析度 | 10-bit | 10-bit |
| Battery sensing channel | ch7（E0）/ ch15（E1） | ch7（E0）/ ch15（E1） |
| 參考電壓 | 2.5V / 1.2V / 外部兩段 | 2.5V / 1.2V / 外部兩段 |
| Compensating method | 有 | 有 |
| Hysteresis | 每 channel 一個 reg | 每 channel 一個 reg |

> 兩者 register 結構完全相同，主要差異為 base address。
> AST2700 ADC 為 BMC SoC 本地 ADC；AST1700 ADC 位於遠端 IO Expander，透過 LTPI AHB bridge 映射至 AST2700 位址空間（`0x30Cxxxxx`）。

---

## 二、Register Offset 對照

Engine 0 base offset：`000h`；Engine 1 base offset：`100h`（兩邊均同）

| Offset（E0 / E1） | 功能 | AST1700 | AST2700 |
|------------------|------|:-------:|:-------:|
| `000h` / `100h` | Engine Control | ✓ | ✓（同） |
| `004h` / `104h` | Interrupt Enable & Status | ✓ | ✓（同） |
| `008h` / `108h` | ADC VGA Detect Control | ✓ | ✓（同） |
| `00Ch` / `10Ch` | ADC Clock Control | ✓ | ✓（同） |
| `010h~01Ch` / `110h~11Ch` | Channel Data（每 reg 含 2 ch） | ✓ | ✓（同） |
| `030h~04Ch` / `130h~14Ch` | Upper/Lower Bound（per ch） | ✓ | ✓（同） |
| `070h~08Ch` / `170h~18Ch` | Hysteresis Control（per ch） | ✓ | ✓（同） |
| `0C0h` / `1C0h` | Interrupt Source Selection | ✓ | ✓（同） |
| `0C4h` / `1C4h` | Compensating & Trimming | ✓ | ✓（同） |
| `0CCh` / `1CCh` | Global Interrupt Status | 名稱列於 index，無 bit 定義 | 名稱列於 index，無 bit 定義 |

---

## 三、Engine Control Register（offset 000h / 100h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:24 | Reserved | ✓ | ✓ |
| 23:16 | Channel enable（bit per ch，1: Enable / 0: Skip） | ✓ | ✓（同） |
| 15:14 | Reserved | ✓ | ✓ |
| 13 | Enable Battery Sensing | ✓ | ✓（同） |
| 12 | Ch7/Ch15 Selection（0: Normal Voltage / 1: Battery） | ✓ | ✓（同） |
| 11:9 | Reserved | ✓ | ✓ |
| 8 | Initial sequence complete（RO） | ✓ | ✓（同） |
| 7:6 | Reference Voltage Selection<br>00b: 2.5V / 01b: 1.2V / 10b: Ext 1.55~2.7V / 11b: Ext 0.9~1.65V | ✓ | ✓（同） |
| 5 | Auto Compensating sensing mode（1: Trigger / 0: Done） | ✓ | ✓（同） |
| 4 | Compensating sensing mode | ✓ | ✓（同） |
| 3:1 | ADC Operation mode（000b: Power down / 001b: Standby / 111b: Normal） | ✓ | ✓（同） |
| 0 | Engine enable | ✓ | ✓（同） |

---

## 四、Interrupt Enable & Status Register（offset 004h / 104h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:24 | Reserved | ✓ | ✓ |
| 23:16 | Interrupt enable（bit per ch，1: Enable / 0: Disable） | ✓ | ✓（同） |
| 15:8 | Reserved | ✓ | ✓ |
| 7:0 | Interrupt status（W1C，bit per ch） | ✓ | ✓（同） |

---

## 五、ADC VGA Detect Control（offset 008h / 108h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:17 | Reserved | ✓ | ✓ |
| 16 | ADC VGA detect enable | ✓ | ✓（同） |
| 15:0 | Divisor of ADC clock for VGA detection<br>Period = PCLK × 2 × (val + 1) | ✓ | ✓（同） |

---

## 六、ADC Clock Control（offset 00Ch / 10Ch）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:16 | Reserved | ✓ | ✓ |
| 15:0 | Divisor of ADC clock<br>Period = PCLK × 2 × (val + 1)，建議 < 6MHz<br>Sample rate = Period × 12 | ✓ | ✓（同） |

---

## 七、Channel Data Register（offset 010h~01Ch / 110h~11Ch）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:26 | Reserved | ✓ | ✓ |
| 25:16 | 高 channel data（10-bit，RO） | ✓ | ✓（同） |
| 15:10 | Reserved | ✓ | ✓ |
| 9:0 | 低 channel data（10-bit，RO） | ✓ | ✓（同） |

每個 register 含 2 個 channel：

| Offset（E0 / E1） | 高 channel | 低 channel |
|------------------|:----------:|:----------:|
| `010h` / `110h` | ch1 / ch9 | ch0 / ch8 |
| `014h` / `114h` | ch3 / ch11 | ch2 / ch10 |
| `018h` / `118h` | ch5 / ch13 | ch4 / ch12 |
| `01Ch` / `11Ch` | ch7 / ch15 | ch6 / ch14 |

---

## 八、Upper/Lower Bound Register（offset 030h~04Ch / 130h~14Ch）

每 channel 一個 register，ch0~ch7 offset 030h~04Ch，ch8~ch15 offset 130h~14Ch。

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:26 | Reserved | ✓ | ✓ |
| 25:16 | Upper bound（10-bit） | ✓ | ✓（同） |
| 15:10 | Reserved | ✓ | ✓ |
| 9:0 | Lower bound（10-bit） | ✓ | ✓（同） |

---

## 九、Hysteresis Control Register（offset 070h~08Ch / 170h~18Ch）

每 channel 一個 register，ch0~ch7 offset 070h~08Ch，ch8~ch15 offset 170h~18Ch。

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31 | Hysteresis enable（1: Enable / 0: Disable） | ✓ | ✓（同） |
| 30:26 | Reserved | ✓ | ✓ |
| 25:16 | Upper bound（10-bit） | ✓ | ✓（同） |
| 15:10 | Reserved | ✓ | ✓ |
| 9:0 | Lower bound（10-bit） | ✓ | ✓（同） |

---

## 十、Interrupt Source Register（offset 0C0h / 1C0h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:8 | Reserved | ✓ | ✓ |
| 7:0 | Interrupt Source per channel<br>1: Primary Processor / 0: Coprocessor | ✓ | ✓（同） |

---

## 十一、Compensating & Trimming Register（offset 0C4h / 1C4h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:26 | Reserved | ✓ | ✓ |
| 25:16 | Compensating value（RO） | ✓ | ✓（同） |
| 15:4 | Reserved | ✓ | ✓ |
| 3:0 | Trimming value（RW） | ✓ | ✓（同） |

---

## 十二、主要差異總結

| 差異點 | AST1700 | AST2700 |
|--------|---------|---------|
| Base Address | `0x30C0_0000` | `0x14C40000` |
| 晶片角色 | IO Expander（遠端，透過 LTPI AHB bridge 存取） | BMC SoC（本地） |
| Register 結構 | 兩者完全相同 | 兩者完全相同 |
| Global Interrupt Status（0CCh/1CCh） | 名稱列於 index，無 bit 定義 | 名稱列於 index，無 bit 定義 |
