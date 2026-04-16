---
title: Edge AI
type: concept
created: 2026-04-16
updated: 2026-04-16
sources: [digital-twin-edge-ai-federated-learning]
tags: [概念, AI, 邊緣運算]
---

# Edge AI

邊緣人工智慧

## 定義

在資料產生的源頭（邊緣裝置）執行 AI 推論，而非將資料傳送到雲端處理。

## 優勢

- **低延遲**：毫秒級回應時間
- **隱私保護**：資料不離開本地
- **頻寬節省**：減少雲端傳輸
- **離線運作**：不依賴網路連接

## 常見邊緣裝置

| 裝置 | 用途 |
|------|------|
| NVIDIA Jetson | 機器人、工業 |
| Raspberry Pi | 原型開發 |
| Google Edge TPU | 視覺 AI |
| Intel NCS | 推論加速 |

## 在數位孿生中的角色

Edge AI 在數位孿生系統中負責：
1. 即時資料處理
2. 異常檢測
3. 本地決策
4. 與 [[Federated Learning]] 結合的分散式訓練

## 常用輕量模型

- MobileNet
- Tiny-YOLO
- EfficientNet-Lite

## 相關概念

- [[Federated Learning]]
- [[數位孿生]]
- [[Co-simulation]]

## 來源引用

- [[sources/digital-twin-edge-ai-federated-learning|Edge AI + FL 框架]]
