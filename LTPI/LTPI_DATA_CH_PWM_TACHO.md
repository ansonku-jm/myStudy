# AST2700 vs AST1700 PWM/TACHO 完整對照

分析來源：
- `AST1700_A1_Datasheet_v10_clean2.pdf` Chapter 17
- `AST2700_A1_Datasheet_v10_clean2_2.pdf` Chapter 62
- `build/ast2700-default/workspace/sources/linux-aspeed/drivers/hwmon/aspeed-g6-pwm-tach.c`
- `build/ast2700-default/workspace/sources/linux-aspeed/arch/arm64/boot/dts/aspeed/aspeed-ltpi0.dtsi`

---

## 一、架構總覽

| 項目 | AST1700 PWM/TACHO | AST2700 PWM/TACHO |
|------|:-----------------:|:-----------------:|
| 晶片角色 | IO Expander（遠端，透過 LTPI AHB bridge 存取）| BMC SoC（本地）|
| Base Address | `0x300C_0000` | `0x140C_0000` |
| DTS compatible | `aspeed,ast2700-pwm-tach` | `aspeed,ast2700-pwm-tach` |
| PWM channel 數 | 16 | 16 |
| TACH channel 數 | 16 | 16 |

> Register 結構完全相同，主要差異為 Base Address。

---

## 二、Register Offset 對照

**Per-channel registers（ch = 0~15，步進 0x10）：**

| Offset | 功能 | AST1700 | AST2700 |
|--------|------|:-------:|:-------:|
| ch×10h + 00h | PWM Control | ✓ | ✓（同） |
| ch×10h + 04h | PWM Duty Cycle | ✓ | ✓（同） |
| ch×10h + 08h | TACH Control | ✓ | ✓（同） |
| ch×10h + 0Ch | TACH Status | ✓ | ✓（同） |

**Group registers：**

| Offset | 功能 | AST1700 | AST2700 |
|--------|------|:-------:|:-------:|
| 100h | PWM clock gating [15:0] | ✓ | ✓（同） |
| 104h | PWM duty gating [15:0] | ✓ | ✓（同） |
| 108h | Reserved | ✓ | ✓（同） |
| 10Ch | TACH interrupt status [15:0] | ✓ | ✓（同） |

---

## 三、PWM Control Register（offset ch×10h + 00h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 19 | LOAD_SEL_RISING_AS_WDT | ✓ | ✓（同） |
| 18 | DUTY_LOAD_AS_WDT_ENABLE | ✓ | ✓（同） |
| 17 | DUTY_SYNC_DISABLE | ✓ | ✓（同） |
| 16 | CLK_ENABLE（0: inactive）| ✓ | ✓（同） |
| 15 | LEVEL_OUTPUT | ✓ | ✓（同） |
| 14 | INVERSE | ✓ | ✓（同） |
| 13 | OPEN_DRAIN_ENABLE | ✓ | ✓（同） |
| 12 | PIN_ENABLE（1: 輸出使能）| ✓ | ✓（同） |
| 11:8 | CLK_DIV_H | ✓ | ✓（同） |
| 7:0 | CLK_DIV_L | ✓ | ✓（同） |

---

## 四、PWM Duty Cycle Register（offset ch×10h + 04h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31:24 | DUTY_CYCLE_PERIOD | ✓ | ✓（同） |
| 23:16 | DUTY_CYCLE_POINT_AS_WDT | ✓ | ✓（同） |
| 15:8 | DUTY_CYCLE_FALLING_POINT | ✓ | ✓（同） |
| 7:0 | DUTY_CYCLE_RISING_POINT | ✓ | ✓（同） |

---

## 五、TACH Control Register（offset ch×10h + 08h）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31 | IER（interrupt enable）| ✓ | ✓（同） |
| 30 | INVERS_LIMIT | ✓ | ✓（同） |
| 29 | LOOPBACK | ✓ | ✓（同） |
| 28 | ENABLE | ✓ | ✓（同） |
| 27:26 | DEBOUNCE（00=3clk / 01=2 / 10=1 / 11=0）| ✓ | ✓（同） |
| 25:24 | IO_EDGE（00=F2F / 01=R2R / 10=BOTH）| ✓ | ✓（同） |
| 23:20 | CLK_DIV_T（除數 = 4^n）| ✓ | ✓（同） |
| 19:0 | THRESHOLD（轉速下限）| ✓ | ✓（同） |

---

## 六、TACH Status Register（offset ch×10h + 0Ch）

| Bit | 功能 | AST1700 | AST2700 |
|-----|------|:-------:|:-------:|
| 31 | ISR（interrupt status）| ✓ | ✓（同） |
| 25 | PWM_OUT | ✓ | ✓（同） |
| 24 | PWM_OEN | ✓ | ✓（同） |
| 23 | DEB_INPUT | ✓ | ✓（同） |
| 22 | RAW_INPUT | ✓ | ✓（同） |
| 21 | VALUE_UPDATE | ✓ | ✓（同） |
| 20 | FULL_MEASUREMENT | ✓ | ✓（同） |
| 19:0 | VALUE（tacho count，計算 RPM 用）| ✓ | ✓（同） |

---

## 七、RPM 計算公式（兩者相同）

`RPM = (AHB_CLK × 60) / [PulsePR × (TachoValue + 1) × TachoClkDiv]`

- PulsePR：每轉 pulse 數（fan dependent，一般為 2）
- TachoClkDiv = 4^CLK_DIV_T

---

## 八、主要差異總結

| 差異點 | AST1700 | AST2700 |
|--------|---------|---------|
| Base Address | `0x300C_0000` | `0x140C_0000` |
| 晶片角色 | IO Expander（透過 LTPI AHB bridge 存取）| BMC SoC（本地）|
| Register 結構 | 兩者完全相同 | 兩者完全相同 |
