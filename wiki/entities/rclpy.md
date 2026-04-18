---
title: rclpy
type: entity
created: 2026-04-16
updated: 2026-04-18
sources: [factory-floor-digital-twin]
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
| `__init__(name)` | 建立節點，name 為節點名稱 | `super().__init__("machine_publisher")` |
| `create_publisher(msg_type, topic, qos)` | 建立發布者 | `self.create_publisher(String, '/factory/machine_01/status', 10)` |
| `create_subscription(msg_type, topic, callback, qos)` | 建立訂閱者 | `self.create_subscription(String, '/topic', self.callback, 10)` |
| `create_timer(period, callback)` | 建立定時器，定期執行回呼 | `self.create_timer(1.0, self.publishMachineData)` |
| `get_logger()` | 取得日誌記錄器 | `self.get_logger().info(f"{machine_id} -> {msg.data}")` |

### Publisher 物件

```python
publisher = self.create_publisher(String, '/topic', 10)
```

| 方法 | 說明 | 範例 |
|------|------|------|
| `publish(msg)` | 發布訊息到 topic | `publisher.publish(msg)` |

### Subscription 物件（create_subscription）

訂閱 topic 並接收訊息的方法，與 `create_publisher` 相對應。

#### 語法

```python
self.create_subscription(
    msg_type,      # 訊息類型（如 String, Float32）
    topic,         # 要訂閱的 topic 名稱
    callback,      # 收到訊息時呼叫的函式
    qos            # 服務品質（佇列深度）
)
```

#### 參數說明

| 參數 | 類型 | 說明 |
|------|------|------|
| `msg_type` | class | 訊息類別，如 `String`, `Float32` |
| `topic` | str | topic 路徑，如 `"/factory/machine_01/status"` |
| `callback` | callable | 收到訊息時執行的回呼函式 |
| `qos` | int | 佇列深度，通常用 `10` |

#### 與 Publisher 的對比

| Publisher（發布者） | Subscriber（訂閱者） |
|---------------------|----------------------|
| `create_publisher()` | `create_subscription()` |
| `publisher.publish(msg)` 主動發送 | `callback(msg)` 被動接收 |

#### 執行流程

```
1. create_subscription() 註冊訂閱
              ↓
2. rclpy.spin(node) 開始監聽
              ↓
3. 有訊息進入 topic
              ↓
4. callback(msg) 自動被呼叫
              ↓
5. 處理訊息（轉發、儲存、顯示等）
```

#### 基本用法

```python
class MySubscriber(Node):
    def __init__(self):
        super().__init__("my_subscriber")

        self.subscription = self.create_subscription(
            String,                        # 訊息類型
            "/factory/machine_01/status",  # topic
            self.on_message,               # 回呼函式
            10                             # qos
        )

    def on_message(self, msg):
        """收到訊息時自動呼叫"""
        self.get_logger().info(f"收到: {msg.data}")
```

#### 動態建立多個訂閱

```python
# 使用 lambda 預設參數捕捉變數
for ros2_topic, mqtt_topic in TOPIC_MAP.items():
    self.create_subscription(
        String,
        ros2_topic,
        lambda msg, t=mqtt_topic: self.onMessage(msg, t),
        10
    )
```

**注意**：在迴圈中使用 lambda 時，必須用預設參數 `t=mqtt_topic` 捕捉當前值，否則所有 callback 都會參照到最後一個值。

#### 回呼函式簽名

```python
def callback(self, msg):
    # msg 是收到的訊息物件
    # 對於 String 類型，用 msg.data 取得字串內容
    data = msg.data
```

## 訊息類型

### std_msgs.msg

```python
from std_msgs.msg import String, Float32
```

| 類型 | 說明 | 欄位 | 用途 |
|------|------|------|------|
| `String` | 字串訊息 | `data: str` | JSON 資料傳輸 |
| `Float32` | 32位元浮點數 | `data: float` | 溫度、振動數值 |
| `Float64` | 64位元浮點數 | `data: float` | 高精度數值 |
| `Int32` | 32位元整數 | `data: int` | 計數器、狀態碼 |
| `Bool` | 布林值 | `data: bool` | 開關狀態 |

### 訊息使用方式

```python
from std_msgs.msg import String

# 建立訊息
msg = String()

# 設定資料
msg.data = json.dumps({"machine_id": "machine_01", "temperature": 75.5})

# 發布
publisher.publish(msg)
```

## QoS（服務品質）

`qos` 參數控制訊息傳遞的可靠性：

| 值 | 說明 |
|----|------|
| `10` | 佇列深度，保留最近 10 筆訊息（常用預設值） |

## 專案實例

### factory-floor-digital-twin

**Publisher 節點**：
```python
# machine_publisher.py
class MachinePublisher(Node):
    def __init__(self):
        super().__init__("machine_publisher")

        # 使用字典推導式建立多個 publisher
        self.publishers_ = {
            machine: self.create_publisher(String, f"/factory/{machine}/status", 10)
            for machine in MACHINES
        }

        # 定時發布
        self.timer = self.create_timer(PUBLISH_INTERVAL, self.publishMachineData)
        self.get_logger().info("MachinePublisher has already activated")
```

**Subscriber 節點（Bridge）**：
```python
# ros2_to_mqtt.py
class Ros2MqttBridge(Node):
    def __init__(self):
        super().__init__("ros2_mqtt_bridge")

        # 動態建立多個 subscription
        for ros2_topic, mqtt_topic in TOPIC_MAP.items():
            self.create_subscription(
                String,
                ros2_topic,
                lambda msg, t=mqtt_topic: self.onRos2Message(msg, t),
                10
            )

    def onRos2Message(self, msg, mqtt_topic):
        # 轉發到 MQTT
        self.mqttClient_.publish(mqtt_topic, msg.data)
```

## 相關實體

- [[ROS2]]
- [[std_msgs]]

## 來源引用

- 2026-04-16：MachinePublisher 練習程式碼
- 2026-04-17：factory-floor-digital-twin 專案分析
- 2026-04-18：Ros2MqttBridge Subscriber 模式
