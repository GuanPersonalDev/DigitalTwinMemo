---
title: ROS2 Python 模式
type: synthesis
created: 2026-04-16
updated: 2026-04-17
sources: [factory-floor-digital-twin]
tags: [程式碼, ROS2, Python, 模式]
---

# ROS2 Python 模式

常見的 ROS2 Python 程式碼模式與範例，基於 factory-floor-digital-twin 專案。

## 基本節點結構

所有 ROS2 Python 節點都遵循這個基本結構：

```python
import rclpy
from rclpy.node import Node

class MyNode(Node):
    def __init__(self):
        super().__init__("node_name")  # 初始化節點，設定名稱
        # 初始化 publisher, subscriber, timer 等

def main(args=None):
    rclpy.init(args=args)      # 1. 初始化 ROS2
    node = MyNode()            # 2. 建立節點實例
    rclpy.spin(node)           # 3. 持續運行節點
    rclpy.shutdown()           # 4. 關閉 ROS2

if __name__ == "__main__":
    main()
```

## 工廠機台 Publisher（完整範例）

來自 `factory-floor-digital-twin/ros2_publisher/machine_publisher.py`：

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
import json
import random

MACHINES = ["machine_01", "machine_02", "machine_03"]
PUBLISH_INTERVAL = 1.0  # second

class MachinePublisher(Node):
    def __init__(self):
        super().__init__("machine_publisher")

        # 字典推導式建立多個 publisher
        self.publishers_ = {
            machine: self.create_publisher(String, f"/factory/{machine}/status", 10)
            for machine in MACHINES
        }

        self.timer = self.create_timer(PUBLISH_INTERVAL, self.publishMachineData)
        self.get_logger().info("MachinePublisher has already activated")

    def generateMachineData(self, machine_id):
        """生成模擬機台數據"""
        return {
            "machine_id": machine_id,
            "temperature": round(random.uniform(60.0, 95.0), 1),
            "vibration": round(random.uniform(0.1, 5.0), 2),
            "status": random.choice(["running", "running", "running", "warning", "error"]),
        }

    def publishMachineData(self):
        """定時發布所有機台數據"""
        for machine_id, publisher in self.publishers_.items():
            data = self.generateMachineData(machine_id)
            msg = String()
            msg.data = json.dumps(data)
            publisher.publish(msg)
            self.get_logger().info(f"{machine_id} -> {msg.data}")

def main(args=None):
    rclpy.init(args=args)
    node = MachinePublisher()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

## 多 Publisher 模式（字典推導式）

使用字典推導式動態建立多個 publisher：

```python
MACHINES = ["machine_01", "machine_02", "machine_03"]

# 方法一：字典推導式（推薦）
self.publishers_ = {
    machine: self.create_publisher(String, f"/factory/{machine}/status", 10)
    for machine in MACHINES
}

# 方法二：迴圈
self._publisher_dic = {}
for i in range(3):
    topic = f'/factory/machine{i}/temperature'
    self._publisher_dic[topic] = self.create_publisher(Float32, topic, 10)
```

## ROS2 → MQTT 橋接模式（完整範例）

來自 `factory-floor-digital-twin/bridge/ros2_to_mqtt.py`：

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
import paho.mqtt.client as mqtt
from config import MQTT_BROKER_HOST, MQTT_BROKER_PORT, TOPIC_MAP

class Ros2MqttBridge(Node):
    def __init__(self):
        super().__init__("ros2_mqtt_bridge")

        # 初始化 MQTT 客戶端
        self.mqttClient_ = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
        self.mqttClient_.connect(MQTT_BROKER_HOST, MQTT_BROKER_PORT)
        self.mqttClient_.loop_start()

        # 動態建立多個 subscription（使用 lambda 捕捉變數）
        for ros2_topic, mqtt_topic in TOPIC_MAP.items():
            self.create_subscription(
                String,
                ros2_topic,
                lambda msg, t=mqtt_topic: self.onRos2Message(msg, t),
                10
            )
        self.get_logger().info("Ros2MqttBridge has already activated")

    def onRos2Message(self, msg, mqtt_topic):
        self.mqttClient_.publish(mqtt_topic, msg.data)
        self.get_logger().info(f"Published: {mqtt_topic} -> {msg.data}")

def main(args=None):
    rclpy.init(args=args)
    node = Ros2MqttBridge()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

## 多 Subscriber 模式（Lambda 技巧）

在迴圈中建立多個 subscription 時，需要用 lambda 預設參數捕捉變數：

```python
# 錯誤寫法（所有 callback 都會參照最後一個 topic）
for ros2_topic, mqtt_topic in TOPIC_MAP.items():
    self.create_subscription(
        String, ros2_topic,
        lambda msg: self.onMessage(msg, mqtt_topic),  # ❌ mqtt_topic 會是最後一個值
        10
    )

# 正確寫法（使用預設參數捕捉當前值）
for ros2_topic, mqtt_topic in TOPIC_MAP.items():
    self.create_subscription(
        String, ros2_topic,
        lambda msg, t=mqtt_topic: self.onMessage(msg, t),  # ✓ t 捕捉當前 mqtt_topic
        10
    )
```

## Topic 命名規範

工廠場景常用的 topic 命名：

```
/factory/{machine_id}/status      # 機台狀態（JSON）
/factory/{machine_id}/temperature # 溫度數值
/factory/{machine_id}/vibration   # 振動數值
```

## 模擬資料生成

### random 模組

```python
import random

# 隨機浮點數（範圍）
temperature = round(random.uniform(60.0, 95.0), 1)  # 60.0 ~ 95.0，1位小數

# 隨機浮點數（保留2位小數）
vibration = round(random.uniform(0.1, 5.0), 2)

# 隨機選擇（加權：running 出現機率較高）
status = random.choice(["running", "running", "running", "warning", "error"])
```

### JSON 序列化

```python
import json

# 編碼（Python dict → JSON string）
data = {
    "machine_id": "machine_01",
    "temperature": 75.5,
    "vibration": 2.3,
    "status": "running"
}
msg.data = json.dumps(data)
# 輸出: '{"machine_id": "machine_01", "temperature": 75.5, ...}'

# 解碼（JSON string → Python dict）
received_data = json.loads(msg.data)
print(received_data["temperature"])  # 75.5
```

## Python 語法技巧

### 字典推導式

```python
# 基本語法
{key: value for item in iterable}

# 實例：建立 publisher 字典
publishers = {
    machine: self.create_publisher(String, f"/factory/{machine}/status", 10)
    for machine in ["machine_01", "machine_02", "machine_03"]
}
```

### f-string 格式化

```python
machine_id = "machine_01"
temperature = 75.5

# topic 名稱
topic = f"/factory/{machine_id}/status"

# 日誌訊息
self.get_logger().info(f"{machine_id} -> {msg.data}")
```

### 字典遍歷

```python
# 遍歷 key-value
for machine_id, publisher in self.publishers_.items():
    publisher.publish(msg)
```

### round() 函式

```python
# 四捨五入到指定小數位
round(75.567, 1)  # 75.6
round(75.567, 2)  # 75.57
```

## 常見匯入

```python
# ROS2 核心
import rclpy
from rclpy.node import Node

# 標準訊息
from std_msgs.msg import String, Float32, Int32, Bool

# 常用工具
import json
import random
from datetime import datetime

# MQTT（橋接用）
import paho.mqtt.client as mqtt
```

## 執行節點

```bash
# 直接執行
python3 machine_publisher.py

# 透過 ROS2 執行（需要設定 package）
ros2 run factory_publisher machine_publisher
```

## 相關頁面

- [[entities/rclpy|rclpy]] - 函式庫詳細說明
- [[entities/ROS2|ROS2]] - 指令參考
- [[entities/MQTT|MQTT]] - MQTT 協定

## 來源引用

- 2026-04-16：MachinePublisher 練習程式碼
- 2026-04-17：factory-floor-digital-twin 專案分析
- 2026-04-18：Ros2MqttBridge 橋接模式
