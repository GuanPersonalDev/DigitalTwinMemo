---
title: rclpy
type: entity
created: 2026-04-16
updated: 2026-04-16
sources: []
tags: [函式庫, ROS2, Python]
---

# rclpy

ROS2 Client Library for Python

## 基本資訊

- **類型**：Python 函式庫
- **用途**：在 Python 中開發 ROS2 節點
- **安裝**：隨 ROS2 安裝，或 `pip install rclpy`

## 模組與類別

### rclpy（核心模組）

```python
import rclpy
```

| 函式 | 說明 | 範例 |
|------|------|------|
| `rclpy.init()` | 初始化 ROS2 通訊環境，必須在建立節點前呼叫 | `rclpy.init(args=args)` |
| `rclpy.spin(node)` | 讓節點持續運行，處理回呼函式 | `rclpy.spin(node)` |
| `rclpy.shutdown()` | 關閉 ROS2 通訊環境，釋放資源 | `rclpy.shutdown()` |

### rclpy.node.Node

```python
from rclpy.node import Node
```

| 方法 | 說明 | 範例 |
|------|------|------|
| `__init__(name)` | 建立節點，name 為節點名稱 | `super().__init__("my_node")` |
| `create_publisher(msg_type, topic, qos)` | 建立發布者 | `self.create_publisher(String, '/topic', 10)` |
| `create_subscription(msg_type, topic, callback, qos)` | 建立訂閱者 | `self.create_subscription(String, '/topic', self.callback, 10)` |
| `create_timer(period, callback)` | 建立定時器，定期執行回呼 | `self.create_timer(1.0, self.timer_callback)` |
| `get_logger()` | 取得日誌記錄器 | `self.get_logger().info('message')` |

## 訊息類型

### std_msgs.msg

```python
from std_msgs.msg import String
```

| 類型 | 說明 | 欄位 |
|------|------|------|
| `String` | 字串訊息 | `data: str` |
| `Int32` | 32位元整數 | `data: int` |
| `Float64` | 64位元浮點數 | `data: float` |
| `Bool` | 布林值 | `data: bool` |

### 自訂訊息

*待補充 - 如何建立自訂訊息類型*

## QoS（服務品質）

`qos` 參數控制訊息傳遞的可靠性：
- `10`：佇列深度，保留最近 10 筆訊息
- *更多 QoS 設定待補充*

## 相關實體

- [[ROS2]]
- [[std_msgs]]

## 來源引用

- 2026-04-16：MachinePublisher 練習程式碼
