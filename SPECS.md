# 技術規格報告
# 智慧電表異常用電偵測系統
## Technical Specification — AIoT Smart Meter Anomaly Detection System

> **課程｜Course：** AIoT  
> **組別｜Team：** 1  
> **指導老師｜Instructor：** 陳煥老師  
> **組員｜Members：**
> - 4112056045 資工三 莊明達  
> - 4112056048 資工三 鄭仲軒  
> - 4112064213 電資三 戴煦  
> **版本｜Version：** v1.0  
> **日期｜Date：** 2026-04

---

## 目錄 Table of Contents

1. [概述 Overview](#1-概述-overview)
2. [問題定義與動機 Problem Definition & Motivation](#2-問題定義與動機-problem-definition--motivation)
3. [相關研究與技術比較 Related Work & Technology Comparison](#3-相關研究與技術比較-related-work--technology-comparison)
4. [演算法規格 Algorithm Specification](#4-演算法規格-algorithm-specification)
5. [資料規格 Data Specification](#5-資料規格-data-specification)
6. [特徵工程規格 Feature Engineering Specification](#6-特徵工程規格-feature-engineering-specification)
7. [系統架構規格 System Architecture Specification](#7-系統架構規格-system-architecture-specification)
8. [模型驗證規格 Model Validation Specification](#8-模型驗證規格-model-validation-specification)
9. [輸出與應用層規格 Output & Application Layer Specification](#9-輸出與應用層規格-output--application-layer-specification)
10. [效能評估指標 Performance Evaluation Metrics](#10-效能評估指標-performance-evaluation-metrics)
11. [部署藍圖 Deployment Roadmap](#11-部署藍圖-deployment-roadmap)
12. [未來展望 Future Work](#12-未來展望-future-work)

---

## 1. 概述 Overview

本系統為一套基於**非監督式機器學習**的 AIoT 電力異常分析平台，旨在自動偵測智慧電表數據中的異常用電行為（尤其是竊電行為）。系統採用 **Isolation Forest（孤立森林）** 演算法，整合完整的端對端資料管線，從原始電力時序資料到 Streamlit 視覺化儀表板，提供一個低硬體需求、無需標記資料的即時偵測解決方案。

**This system is an AIoT electrical power anomaly analysis platform based on unsupervised machine learning, designed to automatically detect abnormal power usage behaviors (particularly electricity theft) from smart meter data. It uses the Isolation Forest algorithm within a complete end-to-end data pipeline — from raw time-series electricity data to a Streamlit visualization dashboard — delivering a real-time detection solution with low hardware requirements and no need for labeled data.**

---

## 2. 問題定義與動機 Problem Definition & Motivation

### 2.1 核心問題 Core Problem

竊電屬於電力**非技術性損失（Non-Technical Loss, NTL）**，佔 NTL 總量超過 **90%**。

| 指標 Metric | 數值 Value | 說明 Description |
|---|---|---|
| 全球年度 NTL 損失 | ~USD $101.2B | 2021 年，138 個國家（Northeast Group） |
| 台灣 AMI 安裝數（預計） | ~423 萬具 | 2026 年預計 |
| 傳統抄表週期 | 每月一次 | 發現異常延遲數週至數月 |
| 人工稽查時效 | 數日至月 | 損失難以追回 |

### 2.2 痛點分析 Pain Points

1. **資料量龐大：** 全台 423 萬智慧電表產生的海量時序資料，人工無法處理。
2. **偵測滯後：** 傳統月抄表模式使異常發現延遲數週，損失已擴大。
3. **規則系統誤報率高：** 人工定義規則難以涵蓋所有竊電手法，導致高誤報與稽查成本。
4. **標記資料缺乏：** 竊電行為標記資料取得困難，監督式模型難以訓練。

### 2.3 解決方案目標 Solution Goals

| 目標 Goal | 說明 Description |
|---|---|
| 即時性 Real-time | 以 AI 取代人工，大幅縮短從資料到告警的時間 |
| 無監督 Unsupervised | 無需標記資料即可訓練模型 |
| 低資源 Low-resource | 適合 Edge Computing 的低運算需求 |
| 可重現 Reproducible | 固定隨機種子確保稽核可重現性 |

---

## 3. 相關研究與技術比較 Related Work & Technology Comparison

### 3.1 方法比較總表 Method Comparison

| 研究 Study | 方法 Method | 需標記資料 Labeled Data | 硬體需求 Hardware | 邊緣適用性 Edge Ready |
|---|---|---|---|---|
| Jokar et al. (2016) | CPBETD (SVM + k-Means) | 需要 | 低 | 可 |
| Buzau et al. (2020) | HNN-NTL (LSTM + MLP → Sigmoid) | 需要 | 高 | 否 |
| Oprea et al. (2021) | SR-CNN + GBM | 不需要 | 高 | 否 |
| **本專題 This Project** | **Isolation Forest** | **不需要** | **低** | **是** |

### 3.2 選用 Isolation Forest 的核心理由 Rationale

- **標記資料稀缺：** 真實竊電標籤幾乎無法取得，有監督模型不具實用性。
- **邊緣運算限制：** 即時部署於電表端（Edge Inference）需要低計算開銷。
- **微妙異常偵測：** 規則系統難以捕捉的細微用電模式變化，Isolation Forest 可有效識別。

---

## 4. 演算法規格 Algorithm Specification

### 4.1 Isolation Forest 核心原理

**核心假設 Core Assumption:**  
異常點在資料分布中具有兩個特性：**(1) 數量稀少 (Few)** 和 **(2) 特徵明顯不同 (Distinct)**，因此比正常點更容易被「孤立」。

**運作機制 Mechanism:**

```
1. 隨機選擇一個特徵 (feature)
2. 在該特徵的值域內隨機選擇一個切割點 (split point)
3. 重複步驟 1-2，遞迴構建孤立樹 (Isolation Tree, iTree)
4. 在樹中，路徑長度 (path length) 越短 → 越易被孤立 → 異常分數越高
5. 對多棵 iTree 的路徑長度取平均值，計算最終異常分數 (Anomaly Score)
```

**數學關係 Mathematical Relationship:**

| 情況 Case | 路徑長度 Path Length | 異常分數 Anomaly Score | 判定 Verdict |
|---|---|---|---|
| 正常值 Normal | 長 (Long) | 低 (Low, closer to 0) | 正常 Normal |
| 異常值 Anomaly | 短 (Short) | 高 (High, closer to 1) | 異常 Anomalous |

### 4.2 演算法優勢 Algorithm Advantages

- **線性時間複雜度：** 訓練和推論的計算複雜度均為 O(n)，適合大規模資料。
- **無需距離計算：** 不依賴歐氏距離或密度估計，避免維度詛咒。
- **天然支持批次處理：** 可對新資料直接評分，無需重新訓練。

---

## 5. 資料規格 Data Specification

### 5.1 資料集 Dataset

| 項目 Item | 規格 Spec |
|---|---|
| 資料集名稱 | UCI ElectricityLoad Dataset |
| 資料來源 | UC Irvine Machine Learning Repository |
| 地區 | 葡萄牙（Portugal） |
| 消費者數量 | 370 戶 (consumers / MT_001 ~ MT_370) |
| 資料頻率 | 15 分鐘一筆（每日 96 筆） |
| 資料格式 | 時序電力消耗（kWh） |

### 5.2 資料前處理規格 Preprocessing Specification

| 步驟 Step | 說明 Description |
|---|---|
| 隨機種子固定 | `random_state` 固定，確保 reproducibility |
| 缺失值處理 | 填補或插值（interpolation） |
| 時間索引解析 | 解析時間戳記，建立 datetime index |
| 正規化 | 依需要進行 Min-Max 或 Z-score 標準化 |

---

## 6. 特徵工程規格 Feature Engineering Specification

### 6.1 時序特徵 Temporal Features

| 特徵名稱 Feature | 類型 Type | 說明 Description |
|---|---|---|
| `lag_1` | Lag Feature | 前一時間步的用電量 |
| `lag_2` | Lag Feature | 前兩時間步的用電量 |
| `lag_n` | Lag Feature | 前 n 時間步的用電量（n 為可配置參數） |
| `rolling_mean_k` | Rolling Statistic | 前 k 個時間步的滾動平均值 |
| `rolling_std_k` | Rolling Statistic | 前 k 個時間步的滾動標準差 |

### 6.2 循環週期特徵 Cyclical Features

| 特徵名稱 Feature | 說明 Description |
|---|---|
| `hour` | 一天中的小時（0–23），捕捉日內規律 |
| `weekday` | 一週中的星期（0=Monday, 6=Sunday），捕捉週內規律 |

### 6.3 特徵工程目標 Objectives

- 建立每位消費者的**baseline 用電行為模型**。
- 使模型能識別相對於歷史基準的統計偏差。
- 捕捉用電在不同時間粒度（小時、日、週）上的週期性規律。

---

## 7. 系統架構規格 System Architecture Specification

### 7.1 三層架構 Three-Tier Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER  資料層                                 │
│                                                                              │
│   INPUT: UCI ElectricityLoad Dataset                                         │
│   └── 370 消費者時序用電資料 → 3-400+ 戶                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PROCESSING LAYER  運算層                               │
│                                                                              │
│   PROCESS (資料前處理)                                                         │
│   ├── 固定 Seed → 確保 Reproducibility                                        │
│   └── 特徵工程：Lag + Rolling Statistics + 時間特徵                            │
│                                                                              │
│   VALIDATE (合成異常注入)                                                      │
│   ├── 依據 Jokar et al. 方法注入異常資料                                        │
│   └── 類型：Spikes (突增) + Drops (驟降)                                       │
│                                                                              │
│   AI MODEL (模型訓練與偵測)                                                    │
│   ├── 演算法：Isolation Forest（非監督式）                                      │
│   ├── 無需標記資料                                                              │
│   └── 路徑短 ↔ 易被隔離 ↔ 異常分數高                                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                          APP LAYER  應用層                                    │
│                                                                              │
│   OUTPUT: Streamlit Dashboard                                                │
│   ├── 異常即時標示於時序圖（紅色告警）                                            │
│   └── 匯出異常事件 Log → 供後續人工稽查使用                                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 技術堆疊 Technology Stack

| 層級 Layer | 組件 Component | 技術 Technology |
|---|---|---|
| 資料層 Data | 資料集 | UCI ElectricityLoad Dataset (CSV) |
| 運算層 Processing | 資料處理 | Python, Pandas, NumPy |
| 運算層 Processing | 模型 | scikit-learn `IsolationForest` |
| 運算層 Processing | 特徵工程 | Pandas (shift, rolling) |
| 應用層 App | 視覺化儀表板 | Streamlit |
| 應用層 App | 圖表繪製 | Matplotlib / Plotly |

---

## 8. 模型驗證規格 Model Validation Specification

### 8.1 合成異常注入 Synthetic Anomaly Injection

由於真實竊電標籤不可得，採用**合成異常注入**方式驗證模型靈敏度。

| 異常類型 Anomaly Type | 說明 Description | 對應竊電行為 Theft Behavior |
|---|---|---|
| **Spike（突增）** | 在特定時段，人為大幅提高用電量 | 未授權大量用電 |
| **Drop（驟降）** | 在特定時段，人為大幅降低記錄用電量 | 電表竄改、繞過計量 |

注入方法依據 **Jokar et al. (2016)** 所描述的異常合成策略。

### 8.2 可重現性保障 Reproducibility Guarantee

- 所有隨機過程（模型訓練、異常注入）均固定 `random_state`（隨機種子）。
- 確保實驗結果可重現，符合稽核需求。

---

## 9. 輸出與應用層規格 Output & Application Layer Specification

### 9.1 Streamlit 儀表板功能 Dashboard Features

| 功能 Feature | 說明 Description |
|---|---|
| 時序圖視覺化 | 顯示完整用電時序曲線 |
| 異常標示 | 以**紅色標記**在時序圖上標示異常事件 |
| 異常分數顯示 | 顯示每個時間點的 Anomaly Score |
| 事件 Log 匯出 | 異常事件可匯出為 CSV / 報表供稽查人員使用 |

### 9.2 效益 Benefits

| 指標 Metric | 傳統方式 Traditional | 本系統 This System |
|---|---|---|
| 從資料到告警時間 | 數週至數月 | **分鐘級（Minutes）** |
| 人工成本 | 高 | **顯著降低** |
| 資料處理規模 | 受限人工 | **全量自動化** |

---

## 10. 效能評估指標 Performance Evaluation Metrics

### 10.1 主要指標 Primary Metrics

| 指標 Metric | 說明 Description | 對應目標 Goal |
|---|---|---|
| **Precision（精確率）** | 判定為異常中，真正是異常的比例 | 降低稽查人員的無效出訪成本 |
| **Recall（召回率）** | 所有真實異常中，被成功偵測的比例 | 最大化損失回收，確保不漏報 |
| **F1-Score** | Precision 與 Recall 的調和平均數 | 整體偵測效能的主要平衡指標 |

### 10.2 業務導向解讀 Business-Oriented Interpretation

- **高 Precision** → 減少稽查人員不必要的出訪（降低稽查成本）
- **高 Recall** → 確保盡可能多的竊電行為被發現（最大化損失回收）
- **F1-Score** → 在精確率與召回率之間取得最佳平衡

---

## 11. 部署藍圖 Deployment Roadmap

### 11.1 邊緣推論部署 Edge Inference Deployment

| 階段 Phase | 說明 Description |
|---|---|
| 雲端訓練 Cloud Training | 使用完整 UCI 資料集在伺服器上訓練模型 |
| 模型壓縮 Model Compression | 進行**模型量化（Quantization）** 與剪枝（Pruning） |
| 邊緣部署 Edge Deploy | 部署至 ARM 架構微控制器（如 ESP32、Raspberry Pi） |
| 本地推論 Local Inference | 在電表端進行本地異常評分，保護隱私並降低頻寬需求 |

### 11.2 隱私與頻寬考量 Privacy & Bandwidth Considerations

- **本地推論**確保用電數據不需上傳雲端，保護用戶隱私。
- 僅回傳異常事件告警，大幅降低網路頻寬需求。

---

## 12. 未來展望 Future Work

### 12.1 特徵工程優化 Feature Engineering Enhancements

| 外部變數 External Variable | 預期效益 Expected Benefit |
|---|---|
| 天氣資料（氣溫、降雨） | 剔除因天氣造成的正常用電波動，提升精確率 |
| 節慶/假日資訊 | 排除節假日特殊用電模式的干擾 |
| 工業/住宅分區別 | 針對不同用電模式建立更精準的基準 |

### 12.2 模型部署優化 Model Deployment Enhancements

- 在真實邊緣設備（如 ESP32、Raspberry Pi）上進行實機測試，驗證可行性與延遲表現。
- 探索**線上學習（Online Learning）** 機制，使模型能隨用電行為變化持續自我更新。
- 引入**多電表聯合分析**，偵測群體性異常（如整棟大樓同步異常）。

### 12.3 系統整合 System Integration

- 整合台灣電力公司（Taipower）AMI 系統的真實資料流。
- 建立自動化稽查工單系統，將告警事件直接串接至稽查人員的行動裝置。

---

## 參考文獻 References

| 編號 | 引用 Citation |
|---|---|
| [1] | 竊電佔 NTL 超過九成之統計來源（依原始提案文獻） |
| [2] | NTL 負面影響說明（依原始提案文獻） |
| [3] | Northeast Group (2021). *Electricity Theft & Non-Technical Losses*. https://northeast-group.com/2021/10/20/electricity-theft-non-technical-losses/ |
| [4] | 工商時報. 台灣智慧電表安裝預測. https://www.ctee.com.tw/news/20260407700101-439901 |
| [5] | Jokar, P. et al. (2016). Electricity Theft Detection in AMI Using Customers' Consumption Patterns. *IEEE Transactions on Smart Grid*. |
| [6] | Buzau, M.M. et al. (2020). Detection of Non-Technical Losses Using Smart Meter Data and Supervised Learning. *IEEE Transactions on Smart Grid*. |
| [7] | Oprea, S.V. et al. (2021). Electricity Fraud Detection Using Machine Learning. *Applied Energy*. |
| [8] | UCI ElectricityLoad Dataset. UC Irvine Machine Learning Repository. |

---

*本規格報告版本 v1.0，整合自：*
- *AIoTFinalProjectProposal_Team1.pdf（提案簡報，14頁）*
- *技術評估報告：基於 Isolation Forest 之智慧電表異常用電偵測分析.pdf*
- *AIoT_Project_Proposal_Presentation_Video.mov（提案簡報影片）*
