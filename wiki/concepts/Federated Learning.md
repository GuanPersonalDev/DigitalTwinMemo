---
title: Federated Learning
type: concept
created: 2026-04-16
updated: 2026-04-16
sources: [digital-twin-edge-ai-federated-learning]
tags: [概念, AI, 分散式學習]
---

# Federated Learning

聯邦學習

## 定義

一種分散式機器學習方法，允許多個節點在不共享原始資料的情況下協同訓練模型。

## 運作方式

```
1. 中央伺服器分發全域模型
2. 各節點使用本地資料訓練
3. 節點只傳送梯度（非原始資料）
4. 中央伺服器聚合更新
5. 更新後的模型再次分發
```

## 聚合策略

- **FedAvg**：加權平均
- **FedProx**：處理異質性
- **異步 FL**：節點可隨時加入

## 隱私保護技術

- **差分隱私（Differential Privacy）**
- **同態加密（Homomorphic Encryption）**

## 在數位孿生中的應用

- 跨工廠協同訓練
- 產品生命週期管理
- 預測性維護模型共享
- 邊緣裝置智慧更新

## 優勢

- 資料隱私保護
- 合規性（GDPR 等）
- 減少頻寬使用
- 利用分散式運算資源

## 相關概念

- [[Edge AI]]
- [[數位孿生]]

## 來源引用

- [[sources/digital-twin-edge-ai-federated-learning|Edge AI + FL 框架]]
