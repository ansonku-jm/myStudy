# AST2700 vs AST1700 SGPIO 完整對照

分析來源：
- `AST1700_A1_Datasheet_v10_clean2.pdf` Chapter 15
- `AST2700_A1_Datasheet_v10_clean2_2.pdf` Chapter 54~56
- `build/ast2700-default/workspace/sources/linux-aspeed/drivers/gpio/gpio-aspeed-sgpio.c`

---

## 一、架構總覽

| 項目 | AST1700 SGPIO | AST2700 SGPIO Master (Ch54) | AST2700 SGPIO Slave (Ch55) | AST2700 SGPIO Monitor (Ch56) |
|------|:---:|:---:|:---:|:---:|
| Base Address | `0x30C0D000` | #0: `0x14C0C000`<br>#1: `0x14C0D000` | `0x14C3C000` | #0: `0x14C0E000`<br>#1: `0x14C0E800` |
| Sets 數 | 2 sets | 2 sets | 1 | 2 |
| 最大 pin 數 | 256 | 256 | 128 in + 128 out | — |
| 角色 | Master（驅動外部 shift register） | Master（同左） | Slave（受外部 SGPIO Master 驅動） | Slave Monitor |

> AST1700 只有一種 SGPIO（Master）；AST2700 拆分為 Master / Slave / Monitor 三種獨立 controller。

---

## 二、Register Offset 對照

| Offset | 功能 | AST1700 SGPIO | AST2700 SGPIO Master | AST2700 SGPIO Slave |
|--------|------|:---:|:---:|:---:|
| `00h` | Config Register | ✓ | ✓（結構完全相同） | ✓（多 3 個 control bits） |
| `04h~10h` | Debug Serial In #0~#3（pin 0~127） | — | ✓ | ✓ |
| `14h` | Debug Serial In #4（pin 128~159） | — | ✓ | — |
| `20h` | 80h Enable Register（bit 3:0） | ✓ | ✓ | — |
| `24h~30h` | Debug Serial Out #0~#3（pin 0~127） | — | ✓ | ✓ |
| `34h` | Debug Serial Out #4（pin 128~159） | — | ✓ | — |
| `3Ch` | Write Protection Register | ✓ | ✓ | ✓ |
| `40h~5Ch` | Interrupt Status #0~#7（pin 0~255） | ✓（8 regs，pin 0~255） | ✓（8 regs，pin 0~255） | ✓（4 regs，pin 0~127） |
| `80h+4N` | Per-pin Control（N = 0~255） | ✓ | ✓ | ✓ |

> AST1700 SGPIO 沒有 Debug Serial In/Out Registers（無逐 pin 值 readback）。

---

## 三、Config Register（offset 00h）比較

| Bit | AST1700 SGPIO00 | AST2700 SGPIO Master（SGPIO000） | AST2700 SGPIO Slave（SGPIOS00） |
|-----|-----------------|----------------------------------|----------------------------------|
| 31:16 | Clock division | 同左 | Clock division |
| 14 | Inverse of load | 同左 | Parallel output protect enable（新增） |
| 13 | Inverse of clock | 同左 | Serial input stop update（新增） |
| 12 | — | 同左 | Serial output stop update（新增） |
| 11:6 | Numbers of SGPIO pins（1~10 bytes） | 同左 | Numbers of SGPIO pins（1~10 bytes） |
| 2 | Inverse of Serial GPIO load | 同左 | Inverse of Serial GPIO load |
| 1 | Inverse of Serial GPIO clock | 同左 | Inverse of Serial GPIO clock |
| 0 | Enable | 同左 | Enable |

> AST2700 SGPIO Master Config Register 與 AST1700 完全相同（已由 `gpio-aspeed-sgpio.c` driver 確認）。
> AST2700 SGPIO Slave 新增 bit 14（parallel output protect）、bit 13（serial input stop update）、bit 12（serial output stop update）。

---

## 四、Per-pin Control Register（offset 80h+4N）比較

| Bit | AST1700 SGPIO_N | AST2700 SGPIO_N（Master） | AST2700 SGPIOS_N（Slave） |
|-----|:---:|:---:|:---:|
| 31 | Write protect | Write protect | Write protect |
| 30:20 | Reserved | Reserved | Reserved |
| 19:18 | — | Parallel output selection<br>（0: csr / 1: parallel-in / 2: serial-in）（**新增**） | — |
| 17:16 | — | Serial output selection<br>（0: csr / 1: parallel-in / 2: serial-in）（**新增**） | — |
| 15 | Reserved | Reserved | Reserved |
| 14 | Parallel input value（RO） | Parallel input value（RO） | Parallel input value（RO） |
| 13 | Serial pin input value（RO） | Serial pin input value（RO） | Serial pin input value（RO） |
| 12 | Interrupt status（RO） | Interrupt status（RO） | Interrupt status（RO） |
| 11 | Parallel input mode（AST1700 only） | —（已移除） | — |
| 10 | Parallel Enable（AST1700 only） | —（已移除） | — |
| 9 | Input mask | Input mask | Input mask |
| 8:7 | Reserved | Reserved | Reserved |
| 6 | Reset Polarity | Reset Polarity | Reset Polarity |
| 5:2 | Interrupt control | Interrupt control | Interrupt control |
| 1 | Parallel output csr value | Parallel output csr value | Parallel output csr value |
| 0 | Serial output csr value | Serial output csr value | Serial output csr value |

**主要差異：**
- AST2700 Master 新增 bit 19:18（Parallel output selection）與 bit 17:16（Serial output selection），輸出來源可從 CSR / parallel-in / serial-in 三路選擇
- AST1700 的 bit 11（Parallel input mode）與 bit 10（Parallel Enable）在 AST2700 中已移除，改由 bit 19:18 / 17:16 的 selection 機制取代
