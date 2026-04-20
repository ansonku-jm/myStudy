# GPIO Controller: AST2700 vs AST1700 雙通道架構深度對照

## 1. GPIO 傳輸機制對照表

在 AST2700 (SCM) 與 AST1700 (HPM) 之間的 GPIO 整合分為兩大機制，分別對應不同的實體路徑與延遲特性：

| 特性項目 | **機制 A：Parallel GPIO (Data Channel)** | **機制 B：LTPI GPIO (NLGPI/NLGPO)** |
| :--- | :--- | :--- |
| **傳輸路徑** | LTPI Data Channel (AHB Tunnel) | **LTPI Dedicated GPIO Channel** (Frame-based) |
| **目標控制核** | AST1700 內建 Parallel GPIO 控制器 | **LTPI Controller 內部硬體邏輯** |
| **SCM 存取位址** | **`0x32C0B000`** (暫存器映射) | **`0x14C34C00`** (LTPI-GPIO 節點) |
| **內部基址** | `0x30C0B000` | N/A (由 Link-layer 硬體處理) |
| **腳位總數** | 212 pins (27 組 + 2 組) | 112 GPI / 112 GPO |
| **延遲表現** | 一般 (Normal Latency) | **極低延遲 (Non-Latency)** |
| **典型應用** | 一般狀態 LED, 板端 ID, 按鈕偵測 | **Power Good, Reset, 關鍵系統狀態同步** |
| **DTS Compatible**| `aspeed,ast1700-gpio` | `aspeed,ast2700-ltpi-gpio` |

---

## 2. Parallel GPIO 規格一致性 (機制 A)

經查閱 AST2700 與 AST1700 規格書，其內部 Parallel GPIO 控制器 IP 為完全同質設計：

*   **腳位分組**：均具備 27 組 (Main) + 2 組 (Core) 的結構。
*   **控制能力**：最大支援 212 個控制腳位。
*   **暫存器結構**：
    *   `0x000`: Data Value Register
    *   `0x004`: Direction Register
    *   `0x008`: Interrupt Enable Register
    *   所有暫存器偏移量與位元定義均 **100% 一致**。

---

## 3. LTPI Non-Latency GPIO (機制 B)

此機制是 AST2700 系統架構中的核心設計，專為跨板端同步低延遲訊號而設計：

*   **實作原理**：GPIO 狀態直接嵌入在 LTPI 幀 (Frame) 的專屬位元位中傳輸，繞過兩端晶片的 AHB 匯流排處理流程。
*   **SCM 側配置**：由 `aspeed,ast2700-ltpi-gpio` 驅動管理。
*   **HPM 側配置**：由 AST1700 的 LTPI 實體層邏輯直接解碼並與本地 NLGPI/NLGPO 腳位連動。

---

## 4. 結論分析

1.  **實體與邏輯分離**：在 SCM (AST2700) 側，必須區分正在存取的是位於 **`0x32C0B000`** 的遠端暫存器，還是位於 **`0x14C34C00`** 的 LTPI 專用控制邏輯。
2.  **IP 同質性**：AST1700 的 Parallel GPIO 控制器 (`0x30C0B000`) 與 AST2700 的 Peripheral 域 GPIO 控制器 (`0x14C0B000`) 是相同的硬體 IP，這確保了軟體層級的相容性。
3.  **系統映射關鍵**：雖然 AST1700 內部基址為 `0x30C0B000`，但 SCM 必須透過映射位址 **`0x32C0B000`** 才能進行透明存取。
