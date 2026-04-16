---
title: ROS2 Python 模式
type: synthesis
created: 2026-04-16
updated: 2026-04-16
sources: []
tags: [程式碼, ROS2, Python, 模式]
---

# ROS2 Python 模式

常見的 ROS2 Python 程式碼模式與範例。

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

## Publisher 節點模式

定期發布資料到 topic：

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
import json

class PublisherNode(Node):
    def __init__(self):
        super().__init__("publisher_node")

        # 建立 publisher
        self.publisher = self.create_publisher(
            String,          # 訊息類型
            '/topic_name',   # topic 名稱
            10               # QoS 佇列深度
        )

        # 建立定時器，每秒執行一次
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        msg = String()
        msg.data = json.dumps({"key": "value"})
        self.publisher.publish(msg)
        self.get_logger().info(f'Published: {msg.data}')
```

## Subscriber 節點模式

監聽 topic 並處理資料：

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SubscriberNode(Node):
    def __init__(self):
        super().__init__("subscriber_node")

        # 建立 subscriber
        self.subscription = self.create_subscription(
            String,              # 訊息類型
            '/topic_name',       # topic 名稱
            self.listener_callback,  # 回呼函式
            10                   # QoS 佇列深度
        )

    def listener_callback(self, msg):
        self.get_logger().info(f'Received: {msg.data}')
```

## 多 Publisher 模式

同時發布到多個 topic（如多台機台）：

```python
class MultiPublisherNode(Node):
    def __init__(self):
        super().__init__("multi_publisher")

        # 建立多個 publisher
        self.publishers = {}
        machine_ids = ["machine_01", "machine_02", "machine_03"]

        for machine_id in machine_ids:
            topic = f'/factory/{machine_id}/status'
            self.publishers[machine_id] = self.create_publisher(
                String, topic, 10
            )

        self.timer = self.create_timer(1.0, self.publish_all)

    def publish_all(self):
        for machine_id, publisher in self.publishers.items():
            msg = String()
            msg.data = json.dumps({
                "machine_id": machine_id,
                "status": "running",
                "temperature": random.uniform(20, 80)
            })
            publisher.publish(msg)
```

## JSON 資料格式

ROS2 常用 JSON 字串傳遞複雜資料：

```python
import json

# 編碼
data = {"id": "machine_01", "value": 42.5}
msg.data = json.dumps(data)

# 解碼
received_data = json.loads(msg.data)
```

## 常見匯入

```python
# 核心
import rclpy
from rclpy.node import Node

# 標準訊息
from std_msgs.msg import String, Int32, Float64, Bool

# 常用工具
import json
import random
from datetime import datetime
```

## 執行節點

```bash
# 直接執行
python3 my_node.py

# 透過 ROS2 執行（需要設定 package）
ros2 run package_name node_name
```

## 相關頁面

- [[rclpy]] - 函式庫詳細說明
- [[ROS2]] - 指令參考

## 來源引用

- 2026-04-16：MachinePublisher 練習程式碼
