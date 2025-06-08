---
title: "虛擬化解決方案評估 (VM)"
tags:
  - VMware
categories:
  - SRE
date: 2024-11-13T08:33:17+08:00
slug: sre-enterprise-virtualization-solution-survey
---

## TL; DR

自從 VMware 被博通 (Broadcom) 併購後，其商業模式已不再是過去的買斷制，而是全面轉向訂閱制。對於未來考慮使用虛擬化技術的 IT 單位的成本結構與規劃都帶來了顯著影響。近期單位正好要幫客戶規劃虛擬化方案，故順便簡單紀錄目前評估的解決方案有哪些。

---

## 現有解決方案

### Nutanix

- 採訂閱制。
- 費用並不比 VMware 便宜。

### VMware

1. **Cloud Foundation**：全面整合解決方案。
2. **vSphere Foundation**：
   - 支援 vSAN。
   - vSAN 價格依核心數計費。
3. **vSphere Standard**：
   - 不支援 vSAN。

### KVM 開源方案

- **PVE (Proxmox Virtual Environment)**：
  - 基於 KVM 的開源虛擬化平台。
  - 由釋出廠商提供收費支援服務。

## 目前常見的組合

- vSphere standard + NetApp （硬體式， 用 storage 的技術去保護硬碟，硬碟本身就有 RAID6 的效果）
- vSphere standard + 超融合（軟體式，如 vSAN，做資料保護就是同一份資料至少會有兩份資料存在在不同伺服器裡面）

## vSphere standard + NetApp

為目前客戶較多使用的選擇

1. **擴充性**：

   - 比超融合架構簡單。

2. **NetApp 的控制器架構**：

   - 一台 NetApp 配備兩個控制器 (Controller)，互相備援。
   - 多座 NetApp 可設置為：
     - **Active-Active 模式**：雙向運行。
     - **Active-Standby 模式**：備援模式。
   - **建議**：一般只需購買一台，因為控制器同時損壞的機率低，常見問題多為硬碟故障。

3. **NetApp 擴充方式**：

   - 容量條件：基於容量大小與硬碟數量。
   - 超出條件時需購買獨立機器。
   - 根據需求選擇規格：
     - **IO 效能**：選擇 SSD。
     - **大空間需求**：選擇 SATA。

4. **VM Host Server 建議**：
   - 最少需配置 3 台伺服器。
