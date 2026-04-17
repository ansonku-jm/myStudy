# LTPI Controller: AST2700 vs AST1700 完整比較

## 1. 系統架構概述

| 項目 | AST2700 (SCM) | AST1700 (HPM) |
|------|--------------|--------------|
| 腳色 | Master，執行 link training | Endpoint，回應 AST2700 |
| Channel 數量 | 2 (LTPI0 / LTPI1) | 1 |
| LTPI0 基址 | `0x14C34000` | `0x30C34000`（從 SCM 側映射） |
| LTPI1 基址 | `0x14C35000` | — |
| PHY region | 基址 + `0x200` | 基址 + `0x200` |
| TOP region | 基址 + `0x800` | 基址 + `0x800` |
| DTS compatible | `aspeed-ltpi` | `aspeed-ast1700-ltpi` |
| DTS `remote-controller` | 無 | 有（kernel driver 以此判斷角色） |

---

## 2. Controller Region（LTPICTRL，base+0x000）

兩者 register 結構完全相同，範圍 `0x000`–`0x1F0`。

主要 registers（末段）：

| Offset | Name | 說明 |
|--------|------|------|
| 0x1BC | ltpi_rx_link_sp_data | RX Link Speed frame data |
| 0x1C0 | ltpi_rx_ad_data0 | Advertise frame data (Byte2–5) |
| 0x1C4 | ltpi_rx_ad_data1 | Advertise frame data (Byte6–9) |
| 0x1C8 | ltpi_rx_ad_data2 | Advertise frame data (Byte10–12) |
| 0x1CC | ltpi_rx_con_or_acc_data0 | Configure/Accept frame (Byte2–5) |
| 0x1D0 | ltpi_rx_con_or_acc_data1 | Configure/Accept frame (Byte6–9) |
| 0x1D4 | ltpi_rx_con_or_acc_data2 | Configure/Accept frame (Byte10–12) |
| 0x1D8 | ltpi_data_ch_cfg0 | Init=0x3；TAG_CHK_EN, ADDR_CHK_EN, WAIT_ACK_TO_EN |
| 0x1DC | ltpi_data_ch_cfg1 | Init=0x9C40；WAIT_ACK_TO |
| 0x1E0 | ltpi_data_ch_invld_dbg0 | Invalid address debug (RO) |
| 0x1E4 | ltpi_data_ch_invld_dbg1 | Invalid data debug (RO) |
| 0x1E8 | ltpi_data_ch_invld_dbg2 | Drop frame counters / invalid info (RO) |
| 0x1EC | ltpi_data_ch_invld_dbg3 | Drop frame address (RO) |
| 0x1F0 | ltpi_rx_no_clk | Init=0x314；RX no-clock counters + enable bits |

---

## 3. PHY Region（LTPIPHY，base+0x200）

**AST2700 與 AST1700 完全相同。**

| Offset | Name | Init | 關鍵欄位 |
|--------|------|------|---------|
| 000h | ltpi_phy_ctrl | 0x0 | bit1=`DDR_MODE`, bit0=`SDR_MODE` |
| 004h | ltpi_pll_ctrl | 0x0 | bit9=`RX_PHY_CLK_INV`, bit8=`TX_PHY_CLK_INV`, bit4=`PLL_SET`, bit[2:0]=`PLL_SELECT` |
| 008h | ltpi_phy_align_ctrl | 0x2 | bit[3:0]=`K28_6_ALIGN_NUM` |
| 014h | ltpi_hw_retry_ctrl0 | 0x0 | `LINK_DET_STUCK_TO_CNT`[31:0] |
| 018h | ltpi_hw_retry_ctrl1 | 0x50000 | `HW_RETRY_TO_CNT`[19:16], `LINK_DET_PLL_SET`[11], `LINK_DET_PLL_SELECT`[10:8], `LINK_DET_DDR_MODE`[1], `LINK_DET_SDR_MODE`[0] |
| 01Ch | ltpi_hw_retry_ctrl2 | 0xA0A | `PHY_CLOSE_TIME`[15:8], `PHY_SWITCH_MODE_TIME`[7:0] |
| 020h | ltpi_hw_retry_cnt0 | 0x0 | Link training 各階段 timeout 計數 (RO) |
| 024h | ltpi_hw_retry_cnt1 | 0x0 | `START_HW_RETRY_FLOW_CNT`[31:0] (RO) |
| 028h | ltpi_pat_chk_cfg | 0x0 | `RX_PAT_CHK_CLR_ERR_CNT`[8], `RX_PAT_CHK_EN`[4], `TX_PAT_EN`[0] |
| 02Ch | ltpi_rx_chk_cnt | 0xFFFF | `RX_PAT_CHK_CNT`[31:0] |
| 030h | ltpi_rx_pat_chk_flag | 0x0 | 各 pattern check 狀態 (RO) |
| 034h | ltpi_prot_key | 0x0 | Unlock PHY registers：寫入 `0x1728AACC` |
| 038h | ltpi_tx_debug | 0x0 | `TX_PHY_DBG_EN`[12], `TX_PHY_DBG_10B`[9:0] |
| 03Ch | ltpi_rx_debug | 0x0 | `RX_PHY_DBG_10B`[9:0] (RO) |
| 040h | ltpi_rx_fifo_rd0 | 0x0 | `DDR_FIFO_RD_DATA`[25:16], `SDR_FIFO_RD_DATA`[9:0] (RO) |
| 048h | ltpi_rx_fifo_wr0 | 0x0 | `DDR_FIFO_WR_DATA`[25:16], `SDR_FIFO_WR_DATA`[9:0] (RO) |
| 050h | ltpi_rx_debug1 | 0x0 | `K28_6_DET_ST`[2:0] (RO) |

### PLL_SELECT 編碼（兩者相同）

| 值 | 速率 |
|----|------|
| 0 | 25 Mbps |
| 1 | 50 Mbps |
| 2 | 100 Mbps |
| 3 | 200 Mbps |
| 4 | 250 Mbps |
| 5 | 400 Mbps |
| 6 | 800 Mbps |
| 7 | 1 Gbps |

---

## 4. TOP Region（LTPITOP，base+0x800）

**AST2700 與 AST1700 完全相同。**

| Offset | Name | Init | 關鍵欄位 |
|--------|------|------|---------|
| 000h | ltpi_lvds_tx_ctrl | 0x00410041 | TX0/TX1 LVDS 驅動控制（DS1, DS0, IPREE, IPREE_EN, PD, PU, OE） |
| 004h | ltpi_lvds_rx_ctrl | 0x00010001 | RX0/RX1 LVDS 接收控制（BIAS_EN, ST, IE） |
| 00Ch | ltpi_strap_val | 0x0 | bit0=`STRAP_1700_EN` (RO) |
| 010h | ltpi_sw_force_en | 0x0 | bit1=`SW_FORCE_LVDS_TX_DS`, bit0=`SW_FORCE_1700_EN` |
| 014h | ltpi_sw_force_val | 0x0 | bit0=`SW_1700_EN` |
| 018h | ltpi_password | 0xBEEFCACA | `REG_LTPI_PASSWORD`[31:0] |
| 01Ch | ltpi_id_setting | 0x0011ABCD | bit24=`DUMMY`, bit[23:16]=`VERSION`, bit[15:0]=`OEM_JD` |
| 020h | ltpi_pn_swap | 0x0 | bit3=`RX1_PN_SWAP`, bit2=`RX0_PN_SWAP`, bit1=`TX1_PN_SWAP`, bit0=`TX0_PN_SWAP` |
| 024h | ltpi_lvds_debug | 0x0 | bit4=`LVDS_RX1_DBG`, bit0=`LVDS_RX0_DBG` (RO) |

---

## 5. 主要差異總結

| 面向 | AST2700 | AST1700 |
|------|---------|---------|
| 腳色 | SCM master（發起 link training） | HPM endpoint（回應） |
| Channel 數 | 2（LTPI0 / LTPI1） | 1 |
| 基址（LTPI0） | `0x14C34000` | `0x30C34000` |
| DTS compatible | `aspeed-ltpi` | `aspeed-ast1700-ltpi` |
| `remote-controller` DTS property | 無 | 有 |
| Kernel SCU strap 清除 | 是（`ltpi_scm_init()` 發起 rescan） | 否（probe 時清除 SCU strap） |
| IRQ 啟用 | 是 | 否 |
| CTRL registers（0x000–0x1F0） | 相同 | 相同 |
| PHY registers（0x000–0x050） | **相同** | **相同** |
| TOP registers（0x000–0x024） | **相同** | **相同** |
