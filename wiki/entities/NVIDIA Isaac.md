---
title: NVIDIA Isaac
type: entity
created: 2026-04-16
updated: 2026-04-16
sources: [nvidia-mega-omniverse-blueprint, bmw-omniverse-enterprise]
tags: [平台, NVIDIA, 機器人]
---

# NVIDIA Isaac

## 基本資訊

- **類型**：機器人開發平台
- **開發者**：NVIDIA
- **官網**：https://developer.nvidia.com/isaac

## 主要元件

### Isaac Sim

基於 [[NVIDIA Omniverse]] 的機器人模擬平台：
- 物理精確的模擬環境
- 合成資料生成
- ROS/ROS2 整合
- 多機器人協調

### Isaac ROS

機器人作業系統加速套件：
- GPU 加速的感知管線
- 與 [[ROS2]] 相容
- 支援 Jetson 邊緣裝置

### Isaac SDK

機器人應用開發工具包：
- 導航、操控、感知模組
- 深度學習推論

## 應用場景

- AMR（自主移動機器人）訓練
- 機械手臂模擬
- 人形機器人開發
- 倉儲物流優化

## 與數位孿生的關係

Isaac Sim 在 Omniverse 數位孿生中扮演機器人大腦訓練的角色：
1. 在虛擬環境中生成合成資料
2. 進行領域隨機化訓練
3. 測試無限場景
4. 部署到真實機器人

## 相關實體

- [[NVIDIA Omniverse]]
- [[ROS2]]
- [[Jetson]]

## 來源引用

- [[sources/nvidia-mega-omniverse-blueprint|Mega Blueprint]]
- [[sources/bmw-omniverse-enterprise|BMW 案例研究]]
