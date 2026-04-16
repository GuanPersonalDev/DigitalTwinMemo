---
title: ROS2
type: entity
created: 2026-04-16
updated: 2026-04-16
sources: []
tags: [工具, 框架, 機器人]
---

# ROS2

Robot Operating System 2

## 基本資訊

- **類型**：機器人作業系統/中介軟體框架
- **用途**：機器人軟體開發、感測器整合、數位孿生通訊
- **官網**：https://docs.ros.org/

## 常用指令

### Topic 相關

```bash
# 列出目前所有 topic
ros2 topic list

# 監聽某個 topic 的內容
ros2 topic echo /chatter

# 查看 topic 的訊息類型
ros2 topic info /chatter

# 發布訊息到 topic
ros2 topic pub /chatter std_msgs/msg/String "data: 'Hello'"
```

### Node 相關

```bash
# 列出目前運行中的 node
ros2 node list

# 查看 node 詳細資訊
ros2 node info /node_name
```

### Interface 相關

```bash
# 查看訊息結構
ros2 interface show std_msgs/msg/String

# 列出所有可用的訊息類型
ros2 interface list
```

### Service 相關

```bash
# 列出所有 service
ros2 service list

# 呼叫 service
ros2 service call /service_name service_type "request_data"
```

### 其他常用

```bash
# 查看 ROS2 環境資訊
ros2 doctor

# 錄製 topic 資料
ros2 bag record /topic_name

# 播放錄製的資料
ros2 bag play bag_file
```

## 核心概念

- **Node**：執行單一功能的程序單元
- **Topic**：節點間非同步通訊的管道
- **Service**：請求-回應式的同步通訊
- **Action**：長時間執行的任務，可回報進度
- **Message**：Topic 中傳遞的資料結構

## 與數位孿生的關係

ROS2 常用於連接真實世界的感測器與數位孿生系統，透過 Topic 將即時數據傳送到 Omniverse 或其他視覺化平台。

## 相關實體

- [[NVIDIA Omniverse]]
- [[Isaac Sim]]

## 來源引用

- 2026-04-16：個人練習筆記
