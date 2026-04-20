# AST1700 周邊 Pin Definition 統計

分析來源：`AST1700_A1_Datasheet_v10_clean2.pdf`（Chapter 2 Pin Description，pages 33~54）

---

## 一、Pin Group 總覽（來自 Datasheet page 32）

| Pin Group 名稱 | Pin 數 |
|---------------|--------|
| GPIOLL-Interface | 32 |
| UART-Interface | 8 |
| GPIONL-Interface（dedicated） | 22 |
| SGPIO Master-Interface | 4 |
| JTAG + DBGUART-Interface | 5 |
| LTPI-Interface | — |
| MISC-Interface | — |
| I3CHV and I3CLV-Interface | 32 |
| I2C-Interface | 32 |
| ADC-Interface | 24 |
| TACH+PWM-Interface | 24 |
| Analog-Power | — |
| Power-and-Ground | — |

---

## 二、各周邊 Output Pin 詳細統計

### 2.1 I2C（16 buses）

| 項目 | 內容 |
|------|------|
| 組數 | 16 buses（I2C0~15） |
| Pin Group 總計 | 32 pins |
| Output Pins | 32（SCL + SDA 均為 open-drain，雙向可 output） |
| Input Pins | 32（雙向） |

**Pin 對應：**

| I2C Buses | GPIO 群 | 功能 |
|-----------|---------|------|
| I2C0~3 | GPIOV0~7 | I2CSCL0~3 / I2CSDA0~3 |
| I2C4~5 | GPIOW0~3 | I2CSCL4~5 / I2CSDA4~5 |
| I2C6~7 | GPIOW4~7 | I2CSCL6~7 / I2CSDA6~7（shared GPONL25~26/GPINL25~26） |
| I2C8~11 | GPIOX0~7 | I2CSCL8~11 / I2CSDA8~11（shared GPONL28~31/GPINL28~31） |
| I2C12~15 | GPIOG2~7, GPIOH0~1 | I2CSCL12~15 / I2CSDA12~15（shared PWM8~15） |

> 說明：I2C SCL/SDA 均為 open-drain，master 模式下兩者皆可 output drive。

---

### 2.2 I3C（16 buses：HVI3C + I3CLV）

| 項目 | 內容 |
|------|------|
| 組數 | 16 buses（HVI3C 0~3, 12~15；I3C 4~11） |
| Pin Group 總計 | 32 pins |
| Output Pins | 32（SCL + SDA 均為 push-pull 雙向可 output） |
| Input Pins | 32（雙向） |

**Pin 對應：**

| I3C Buses | GPIO 群 | 功能 |
|-----------|---------|------|
| HVI3C 0~3 | GPIOL0~7 | HVI3CSCL0~3 / HVI3CSDA0~3 |
| I3C 4~7 | GPIOJ0~7 | I3CSCL4~7 / I3CSDA4~7 |
| I3C 8~11 | GPIOK0~7 | I3CSCL8~11 / I3CSDA8~11 |
| HVI3C 12~15 | GPIOI0~7 | HVI3C12~15 SCL / SDA（shared GPONL0~3/GPINL0~3） |

---

### 2.3 ADC（16 channels）

| 項目 | 內容 |
|------|------|
| 組數 | 16 channels（Engine0: ADC0~7，Engine1: ADC8~15） |
| Pin Group 總計 | 24 pins |
| Output Pins | 0（純 analog input） |
| Input Pins | 16（ADC0~15 analog input） |
| 其他 8 pins | AGND / VREF / AVDD 等 power/reference pins |

**Pin 對應：**

| ADC Channel | GPIO 群 | 備註 |
|-------------|---------|------|
| ADC0~7 | GPIY0~7 | 專用 ADC pins |
| ADC8~15 | GPIZ0~7 | Shared GPINL11~21, 27 |

---

### 2.4 PWM（16 channels）

| 項目 | 內容 |
|------|------|
| 組數 | 16 channels（PWM0~15） |
| Output Pins | 16（純 output） |
| Input Pins | 0 |

**Pin 對應：**

| PWM Channel | GPIO 群 | 共用功能 |
|-------------|---------|---------|
| PWM0~7 | GPIOD0~7 | Shared GPONL32~39（在 TACH+PWM group） |
| PWM8~15 | GPIOG2~7, GPIOH0~1 | Shared I2CSCL/SDA12~15（在 I2C group） |

> 說明：TACH+PWM-Interface 24 pins = TACH0~15（16）+ PWM0~7（8）；PWM8~15 計入 I2C group。

---

### 2.5 TACH / FAN Tachometer（16 channels）

| 項目 | 內容 |
|------|------|
| 組數 | 16 channels（TACH0~15） |
| Pin Group 總計 | 16 pins（與 PWM0~7 合計 24 pins 在同一 group） |
| Output Pins | 0（純 input，測量風扇轉速） |
| Input Pins | 16（TACH0~15） |

**Pin 對應：**

| TACH Channel | GPIO 群 | 共用功能 |
|-------------|---------|---------|
| TACH0~7 | GPIOB0~7 | Shared GPINL32~39 |
| TACH8~15 | GPIOC0~7 | Shared GPINL40~47 |

---

### 2.6 UART（2 channels）

| 項目 | 內容 |
|------|------|
| 組數 | 2 channels（UART0 + UART1） |
| Pin Group 總計 | 8 pins |
| Output Pins | 4（UARTTXD0, UARTRTS0, UARTTXD1, UARTRTS1） |
| Input Pins | 4（UARTRXD0, UARTCTS0, UARTRXD1, UARTCTS1） |

**Pin 對應：**

| Signal | GPIO | Ball | 方向 |
|--------|------|------|------|
| UARTCTS0 | GPIOE0 | M2 | IN |
| UARTRTS0 | GPIOE5 | N2 | OUT |
| UARTTXD0 | GPIOE6 | N3 | OUT |
| UARTRXD0 | GPIOE7 | L5 | IN |
| UARTCTS1 | GPIOF0 | P2 | IN |
| UARTRTS1 | GPIOF5 | P3 | OUT |
| UARTTXD1 | GPIOF6 | R1 | OUT |
| UARTRXD1 | GPIOF7 | P4 | IN |

---

### 2.7 GPIO Low Latency（GPIOLL）

| 項目 | 內容 |
|------|------|
| 組數 | 16 output group + 16 input group |
| Pin Group 總計 | 32 pins |
| Output Pins | 16（GPOLL0~15） |
| Input Pins | 16（GPILL0~15） |

**Pin 對應（部分）：**

| Signal | 說明 |
|--------|------|
| GPOLL0~15 | GPIO Low Latency Output，16 pins |
| GPILL0~15 | GPIO Low Latency Input，16 pins |

> 說明：Low Latency GPIO 為獨立專用 IO，不與其他功能共用 pin。

---

### 2.8 GPIO Normal Latency（GPIONL）

| 項目 | 內容 |
|------|------|
| Dedicated GPIONL group | 22 pins |
| Output Pins（dedicated） | ~14 GPONL pins |
| Input Pins（dedicated） | ~8 GPINL pins |

**Dedicated GPIONL group pins（與其他周邊無共用）：**

| GPONL Signal | GPIO | 備註 |
|-------------|------|------|
| GPONL9 | GPIOAA0 | output |
| GPONL10 | GPIOAA3 | output |
| GPONL11 | GPIOE1 | output |
| GPONL12 | GPIOE3 | output |
| GPONL13 | GPIOF1 | output |
| GPONL14 | GPIOF3 | output |
| GPONL19 | GPIOP3 | output |
| GPONL20 | GPIOP4 | output |
| GPONL21 | GPIOP5 | output |
| GPONL22 | GPIOS5 | output |
| GPONL27 | GPIOS0 | output |
| GPONL40 | GPIOE2 | output |
| GPONL41 | GPIOE4 | output |
| GPONL42 | GPIOF2 | output |
| GPONL43 | GPIOP6 | output |
| GPONL44 | GPIOP0 | output |
| GPONL45 | GPIOP1 | output |
| GPONL46 | GPIOP2 | output |
| GPONL47 | GPIOS1 | output |
| GPINL9 | GPIOAA1 | input |
| GPINL10 | GPIOAA4 | input |
| GPINL22 | GPIOS4 | input |

> 說明：除 dedicated 22 pins 外，尚有大量 GPONL/GPINL 與 I2C / I3C / PWM / TACH / ADC 共用（Multi-Function Pin）。

---

## 三、Output Pin 數量快速對照

| 周邊 | 組數 | Output Pins | 備註 |
|------|------|------------|------|
| I2C | 16 buses | 32 pins | SCL + SDA open-drain 雙向 |
| I3C | 16 buses | 32 pins | SCL + SDA push-pull 雙向 |
| ADC | 16 channels | **0** | 純 analog input |
| PWM | 16 channels | 16 pins | 純 output |
| TACH（FAN） | 16 channels | **0** | 純 input，測轉速 |
| UART | 2 channels | 4 pins | TXD×2 + RTS×2 |
| GPIO Low Latency | 16+16 | 16 pins | GPOLL0~15 |
| GPIO Normal Latency | dedicated 22 pins | ~19 pins | GPONL（含部分共用） |

---

## 四、TODO

- [ ] **I2C Pin Assignment 分配確認**
  - AST1700 共 16 buses I2C，其中：
    - **6 buses** 走 LTPI I2C Channel（由 LTPI 硬體直接隧道，不經過 Data Channel）
    - **10 buses** 可走 LTPI Data Channel（AHB bridge 方式存取）
  - 待確認：這 16 buses 的 pin 如何對應到上述兩種 channel？
    - 哪 6 buses 對應到 LTPI I2C Channel？（是 I2C0~5？或根據 LTPI controller 設定決定？）
    - 哪 10 buses 走 Data Channel？pin 與 AHB 位址的對應關係為何？
  - 參考：可查 LTPI I2C Channel spec 以及 AST1700 LTPI controller register 中的 I2C channel mapping 設定

---

## 五、Multi-Function Pin 共用關係

```
I2CSCL/SDA12~15 ─────── 同時為 PWM8~15（GPIOG2~7, GPIOH0~1）
                    同時為 GPONL4~8 / GPINL5~8

HVI3C12~15 SCL/SDA ──── 同時為 GPONL0~3 / GPINL0~3（GPIOI0~7）

HVI3C0~3 SCL/SDA ──────同時為 GPONL15~16,23~24 / GPINL15~16,23~24（GPIOL0~7）

I2C6~7 SCL/SDA ────────同時為 GPONL25~26 / GPINL25~26（GPIOW4~7）

I2C8~11 SCL/SDA ───────同時為 GPONL28~31 / GPINL28~31（GPIOX0~7）

PWM0~7 ─────────────── 同時為 GPONL32~39（GPIOD0~7）

TACH0~7 ────────────── 同時為 GPINL32~39（GPIOB0~7）

TACH8~15 ───────────── 同時為 GPINL40~47（GPIOC0~7）

ADC8~15 ────────────── 同時為 GPINL11~21,27（GPIZ0~7）
```
