---
title: factory-floor-digital-twin 專案
type: source
created: 2026-04-17
updated: 2026-04-17
original: https://github.com/GuanPersonalDev/factory-floor-digital-twin
tags: [專案, ROS2, 數位孿生, Python]
---

# factory-floor-digital-twin 專案

**作者**：GuanPersonalDev
**類型**：個人專案
**來源**：https://github.com/GuanPersonalDev/factory-floor-digital-twin

## 專案概述

工廠地板數位孿生專案，使用 ROS2 模擬工廠機台感測器資料發布。

## 檔案結構

```
factory-floor-digital-twin/
├── python/
│   └── factory_publisher.py    # 早期版本（未完成）
├── ros2_publisher/
│   └── machine_publisher.py    # 完整實作
└── README.md
```

## 核心程式碼分析

### machine_publisher.py

**功能**：模擬 3 台機台的感測器資料，定時發布到 ROS2 topic

**架構**：
```
MachinePublisher (Node)
├── publishers_: Dict[str, Publisher]  # 3 個 publisher
├── timer: Timer                        # 定時器（1秒）
├── generateMachineData()               # 生成模擬資料
└── publishMachineData()                # 發布資料
```

**Topic 結構**：
```
/factory/machine_01/status  →  {"machine_id", "temperature", "vibration", "status"}
/factory/machine_02/status  →  ...
/factory/machine_03/status  →  ...
```

**模擬資料範圍**：
| 欄位 | 類型 | 範圍 |
|------|------|------|
| temperature | float | 60.0 ~ 95.0 |
| vibration | float | 0.1 ~ 5.0 |
| status | string | running / warning / error |

## 使用的 API

### ROS2 / rclpy

| API | 用途 |
|-----|------|
| `rclpy.init()` | 初始化 ROS2 |
| `rclpy.spin()` | 運行節點 |
| `rclpy.shutdown()` | 關閉 ROS2 |
| `Node.__init__()` | 建立節點 |
| `create_publisher()` | 建立發布者 |
| `create_timer()` | 建立定時器 |
| `get_logger().info()` | 日誌記錄 |
| `publisher.publish()` | 發布訊息 |

### Python 標準庫

| 模組 | 函式 | 用途 |
|------|------|------|
| `json` | `dumps()` | 字典轉 JSON 字串 |
| `random` | `uniform()` | 隨機浮點數 |
| `random` | `choice()` | 隨機選擇 |

## Python 語法技巧

| 技巧 | 範例 |
|------|------|
| 字典推導式 | `{m: create_publisher(...) for m in MACHINES}` |
| f-string | `f"/factory/{machine}/status"` |
| `.items()` 遍歷 | `for k, v in dict.items()` |
| `round()` | `round(random.uniform(60, 95), 1)` |

## 執行方式

```bash
# 直接執行
cd ros2_publisher
python3 machine_publisher.py

# 監聽輸出
ros2 topic echo /factory/machine_01/status
```

## 與數位孿生的關係

此專案是數位孿生系統的**資料層**，負責：
1. 模擬真實機台感測器
2. 透過 ROS2 topic 發布資料
3. 為 Omniverse 或其他視覺化平台提供資料來源

**整合架構**：
```
machine_publisher.py (ROS2)
       ↓
    Topic
       ↓
Omniverse Isaac Sim / 其他訂閱者
       ↓
   數位孿生視覺化
```

## 待改進

1. 增加更多感測器類型（電流、壓力等）
2. 加入異常狀態的模擬邏輯
3. 與 Omniverse 整合測試
4. 加入 Subscriber 節點接收控制指令

## 相關頁面

- [[rclpy]] - ROS2 Python 函式庫
- [[ROS2]] - 指令參考
- [[synthesis/ROS2-Python-模式|ROS2 Python 模式]] - 程式碼模式
- [[數位孿生]] - 概念說明
