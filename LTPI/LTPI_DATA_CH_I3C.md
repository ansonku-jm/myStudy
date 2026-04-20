# I3C Controller: AST2700 vs AST1700 深度技術對照報告

## 1. 系統硬體規格概述

AST2700 與 AST1700 內建的 I3C 控制器均基於 **MIPI I3C HCI v1.0/v1.1** 標準實作，提供高度一致的編程介面。

| 特性項目 | AST2700 (SCM) | AST1700 (HPM) | 一致性 |
| :--- | :--- | :--- | :--- |
| **控制器 IP** | MIPI I3C HCI | MIPI I3C HCI | **一致** |
| **DTS Compatible**| `aspeed-i3c-hci` | `aspeed-i3c-hci` | **一致** |
| **Instance 數量** | 16 Sets (Device 0~15) | 16 Sets (Device 0~15) | **一致** |
| **本地基址 (Local)** | `0x14C20000` | `0x30C20000` | N/A |
| **映射基址 (Mapping)** | N/A | **`0x32C20000`** | N/A |
| **Max Bus Speed** | 12.5 MHz (SDR) | 12.5 MHz (SDR) | **一致** |
| **HDR Mode** | 支援 HDR-DDR | 支援 HDR-DDR | **一致** |
| **DMA/SG 支援** | 支援 Scatter-Gather | 支援 Scatter-Gather | **一致** |

---

## 2. 暫存器區塊 (Register Blocks) 詳細對照

每個 I3C 實例的 4KB 空間劃分如下，經查閱兩晶片規格書，所有位元定義均**完全一致**。

| 偏移 (Offset) | 區塊名稱 | 功能深度說明 | AST2700 / AST1700 |
| :--- | :--- | :--- | :--- |
| `0x000-0x03F` | **HCI Capability** | 唯讀暫存器，定義 HCI 版本、FIFO 深度、與 SG/HDR 能力宣告。 | **完全一致** |
| `0x040-0x0BF` | **HCI PIO** | 命令/回應/資料隊列 (Queue) 控制，支援 PIO 模式下的傳輸中斷強制位元。 | **完全一致** |
| `0x100-0x1FF` | **HCI DAT** | **Device Address Table**：儲存 16 個裝置的位址、動態位址與 I3C/I2C 屬性。 | **完全一致** |
| `0x200-0x2FF` | **HCI DCT** | **Device Char Table**：儲存裝置 PID (48-bit)、BCR (Bus) 與 DCR (Device) 特性。 | **完全一致** |
| `0x300-0x3FF` | **HCI RHS** | **Reset / Halt / Status**：控制主機控制器的重置序列與當前運行狀態。 | **完全一致** |
| `0x400-0x5FF` | **I3C Control** | 包含中斷彙整 (Interrupt Aggregation)、重試計數與 Master 角色切換邏輯。 | **完全一致** |
| `0x600-0x6FF` | **I3C Phy Ctrl** | **物理層核心**：定義 SCL 時序 (High/Low period) 與 Open-drain 驅動特性。 | **完全一致** |
| `0x700+` | **HCI Ext Cap** | 擴展能力指標，用於宣告額外的供應商特定功能或進階 DMA 配置。 | **完全一致** |

---

## 3. 關鍵暫存器位元 (Bit-level) 深度對照

以下選取三個關鍵暫存器，展示 AST2700 與 AST1700 的 100% 同質性：

### 3.1 HCI_CAPABILITIES (Offset 0x00C)
*   **Bit 30 (SG_CAPABILITY_DC_EN)**: 兩者皆為 `1b1`，支援 Scatter-Gather 數據通道。
*   **Bit 6 (HDR_DDR_EN)**: 兩者皆為 `1b1`，支援 High Data Rate DDR 模式。
*   **Bit 5 (STANDBY_CR_CAP)**: 兩者皆為 `1b1`，具備作為 Standby Controller 的能力。

### 3.2 DAT_SECTION_OFFSET (Offset 0x030)
*   **Bit 31:28 (DAT_ENTRY_SIZE)**: 兩者皆為 `0x0` (代表 2 DWORDs/entry)。
*   **Bit 11:0 (DAT_TABLE_OFFSET)**: 兩者皆指向 `0x100`。

### 3.3 I3C_PHY_CTRL (Offset 0x600 - 0x610)
這部分在 LTPI 整合中至關重要，因為長距離走線需要精確的時序調整：
*   **SCL_I3C_OD_TIME**: 控制 Open-drain 模式下的 SCL 高低準位時間，兩者暫存器結構一致。
*   **SCL_I3C_PP_TIME**: 控制 Push-pull 模式下的時序，兩者範圍與步進均相同。
*   **INTERNAL_PULL_UP**: 控制內部上拉阻值，對訊號完整性至關重要，兩者定義一致。

---

## 4. 總結與開發建議 (LTPI 應用視角)

1.  **完全同質性**：經由位元級對比，確認 AST2700 與 AST1700 的 I3C 控制器在硬體電路與編程邏輯上是**完全相同的複製品**。
2.  **驅動簡化**：由於 `compatible = "aspeed-i3c-hci"` 完全一致，OpenBMC 的驅動程式不需判斷當前是 AST2700 還是遠端的 AST1700。
3.  **LTPI 隧道透明度**：位於 AST1700 上的 I3C 控制器在 AST2700 系統中透過映射位址 **`0x32C20000`** 呈現。由於暫存器定義完全相同，開發者可以使用與操作本地 I3C 控制器完全一致的配置碼來優化遠端 HPM 上的 I3C 訊號時序。
