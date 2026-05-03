---
title: Python 語法技巧
type: synthesis
created: 2026-04-17
updated: 2026-04-22
sources: [factory-floor-digital-twin]
tags: [程式碼, Python, 語法, 類型提示, 執行緒]
---

# Python 語法技巧

在 ROS2 和數位孿生開發中常用的 Python 語法技巧。

## 推導式

### 字典推導式

動態建立字典，常用於批量建立 ROS2 publisher：

```python
# 語法
{key_expr: value_expr for item in iterable}

# 實例：建立多個 publisher
MACHINES = ["machine_01", "machine_02", "machine_03"]

publishers = {
    machine: self.create_publisher(String, f"/factory/{machine}/status", 10)
    for machine in MACHINES
}
# 結果：{'machine_01': Publisher, 'machine_02': Publisher, ...}
```

### 串列推導式

```python
# 生成數值列表
temperatures = [round(random.uniform(60, 95), 1) for _ in range(10)]
```

## 字串格式化

### f-string（推薦）

Python 3.6+ 最簡潔的格式化方式：

```python
machine_id = "machine_01"
temperature = 75.5

# 變數插入
topic = f"/factory/{machine_id}/status"
# 結果：'/factory/machine_01/status'

# 表達式
msg = f"溫度：{temperature:.1f}°C"
# 結果：'溫度：75.5°C'

# 日誌
self.get_logger().info(f"{machine_id} -> {msg.data}")
```

### 格式規範

| 規範 | 說明 | 範例 | 結果 |
|------|------|------|------|
| `:.1f` | 1 位小數 | `f"{3.14159:.1f}"` | `'3.1'` |
| `:.2f` | 2 位小數 | `f"{3.14159:.2f}"` | `'3.14'` |
| `:>10` | 右對齊 10 字元 | `f"{'hi':>10}"` | `'        hi'` |
| `:04d` | 補零 4 位整數 | `f"{7:04d}"` | `'0007'` |

## 字典操作

### 遍歷

```python
publishers = {"m1": pub1, "m2": pub2, "m3": pub3}

# 遍歷 key
for machine_id in publishers:
    print(machine_id)

# 遍歷 key-value（推薦）
for machine_id, publisher in publishers.items():
    publisher.publish(msg)

# 遍歷 values
for publisher in publishers.values():
    publisher.publish(msg)
```

### 檢查 key 存在

```python
if machine_id in publishers:
    publishers[machine_id].publish(msg)

# 安全取值
publisher = publishers.get(machine_id, None)
```

## 隨機數生成

### random 模組

```python
import random

# 隨機浮點數（範圍）
temp = random.uniform(60.0, 95.0)  # 60.0 <= temp <= 95.0

# 隨機整數
count = random.randint(1, 100)  # 1 <= count <= 100

# 隨機選擇（單一）
status = random.choice(["running", "warning", "error"])

# 加權選擇（Python 3.6+）
status = random.choices(
    ["running", "warning", "error"],
    weights=[80, 15, 5],  # 機率比例
    k=1
)[0]

# 隨機選擇（多個）
samples = random.sample(["a", "b", "c", "d"], k=2)  # 不重複選 2 個
```

## 數值處理

### round() 函式

```python
# 四捨五入到指定小數位
round(75.567, 1)   # 75.6
round(75.567, 2)   # 75.57
round(75.567)      # 76（整數）
round(75.567, 0)   # 76.0（浮點數）
round(75.567, -1)  # 80.0（十位）
```

## JSON 處理

### json 模組

```python
import json

# Python dict → JSON string
data = {"machine_id": "m1", "temperature": 75.5}
json_str = json.dumps(data)
# '{"machine_id": "m1", "temperature": 75.5}'

# JSON string → Python dict
data = json.loads(json_str)
print(data["temperature"])  # 75.5

# 格式化輸出（除錯用）
print(json.dumps(data, indent=2, ensure_ascii=False))
```

## 類別與繼承

### 基本繼承

```python
from rclpy.node import Node

class MachinePublisher(Node):
    def __init__(self):
        # 呼叫父類別建構函式
        super().__init__("machine_publisher")

        # 初始化成員變數
        self.publishers_ = {}
        self.timer = None
```

### 私有方法命名

```python
class FactoryPublisher(Node):
    def __init__(self):
        super().__init__('factory_publisher')
        self._publisher_dic = {}  # 私有變數（慣例）

    def _init_machine_publisher(self, index):  # 私有方法（慣例）
        pass
```

## Lambda 與閉包

### 迴圈中的 Lambda 陷阱

在迴圈中使用 lambda 時，變數會被延遲綁定（late binding），導致所有 lambda 都參照最後一個值。

```python
# ❌ 錯誤：所有 callback 的 topic 都是最後一個
for topic in topics:
    callbacks.append(lambda msg: process(msg, topic))

# ✓ 正確：使用預設參數捕捉當前值
for topic in topics:
    callbacks.append(lambda msg, t=topic: process(msg, t))
```

### 實際應用（ROS2 多訂閱）

```python
# 動態建立多個 subscription
for ros2_topic, mqtt_topic in TOPIC_MAP.items():
    self.create_subscription(
        String,
        ros2_topic,
        lambda msg, t=mqtt_topic: self.onMessage(msg, t),  # t 捕捉當前值
        10
    )
```

**原理**：預設參數在函式定義時求值，而非呼叫時求值。

## 組態模組模式

將設定集中到單獨的 config.py：

```python
# config.py
MQTT_BROKER_HOST = "10.255.255.254"
MQTT_BROKER_PORT = 1883

TOPIC_MAP = {
    "/factory/machine_01/status": "factory/machine_01/status",
    "/factory/machine_02/status": "factory/machine_02/status",
}

# main.py
from config import MQTT_BROKER_HOST, MQTT_BROKER_PORT, TOPIC_MAP
```

## 常數定義

```python
# 模組層級常數（大寫）
MACHINES = ["machine_01", "machine_02", "machine_03"]
PUBLISH_INTERVAL = 1.0  # seconds
DEFAULT_QOS = 10
```

## 類型提示（Type Hints）

### 基本語法（Python 3.9+）

```python
# 參數與回傳值類型
def greet(name: str) -> str:
    return f"Hello, {name}"

# 泛型容器（Python 3.9+ 內建支援）
topics: list[str] = ["topic1", "topic2"]
mapping: dict[str, int] = {"a": 1, "b": 2}

# Python 3.8 以下需要從 typing 匯入
from typing import List, Dict
topics: List[str] = ["topic1", "topic2"]
```

### typing 模組

```python
from typing import Callable, Optional

# Optional：可為 None
callback: Optional[Callable] = None  # 等同 Callable | None

# Callable：可呼叫物件
def set_callback(fn: Callable[[str, dict], None]):
    """fn 接受 (str, dict) 參數，回傳 None"""
    self.callback = fn

# 實際使用
self.messageCallback_: Optional[Callable] = None

def setMessageCallback(self, callback: Callable):
    self.messageCallback_ = callback
```

### 類別變數類型提示

```python
class BaseMqttExtension:
    MQTT_HOST: str = "localhost"
    MQTT_PORT: int = 1883
    MQTT_TOPICS: list[str] = []  # 泛型語法
```

## 執行緒安全

### threading.Lock

```python
import threading

class ThreadSafeClass:
    def __init__(self):
        self.data_ = {}
        self.lock_ = threading.Lock()

    def write(self, key, value):
        with self.lock_:  # 進入時取得鎖，離開時釋放
            self.data_[key] = value

    def read_and_clear(self):
        with self.lock_:
            result = dict(self.data_)  # 淺複製
            self.data_.clear()
        return result
```

### 字典複製

```python
# 淺複製（只複製第一層）
original = {"a": 1, "b": [1, 2]}
copy1 = dict(original)      # 方法一
copy2 = original.copy()     # 方法二
copy3 = {**original}        # 方法三（解包）

# 深複製（完整複製巢狀結構）
import copy
deep = copy.deepcopy(original)
```

## 抽象方法模式

### NotImplementedError

```python
class BaseClass:
    def process(self, data):
        """子類別必須實作"""
        raise NotImplementedError

class ConcreteClass(BaseClass):
    def process(self, data):
        # 實際實作
        return data * 2
```

### Hook 方法模式

```python
class BaseExtension:
    def on_startup(self):
        self.setup()
        self.onStartupHook()  # Hook 給子類別

    def onStartupHook(self):
        pass  # 預設空實作

class MyExtension(BaseExtension):
    def onStartupHook(self):
        print("子類別初始化")
```

## 安全屬性檢查

### hasattr()

```python
# 檢查屬性是否存在
if hasattr(self, 'client_') and self.client_:
    self.client_.disconnect()

# 等同於
try:
    if self.client_:
        self.client_.disconnect()
except AttributeError:
    pass
```

### getattr() 與預設值

```python
# 取得屬性，不存在時回傳預設值
value = getattr(obj, 'attr', default_value)
```

## 取得類別資訊

```python
class MyClass:
    def log(self, msg):
        # 取得類別名稱
        name = self.__class__.__name__  # "MyClass"
        print(f"[{name}] {msg}")

# 繼承時自動反映子類別名稱
class ChildClass(MyClass):
    pass

child = ChildClass()
child.log("hello")  # [ChildClass] hello
```

## 字典安全操作

### .get() 方法

```python
# 安全取值（key 不存在時回傳預設值）
status = data.get("status", "unknown")

# 巢狀字典安全取值
value = data.get("level1", {}).get("level2", default)

# 對比直接存取（會拋出 KeyError）
status = data["status"]  # 若不存在會報錯
```

### in 檢查

```python
if "key" in my_dict:
    value = my_dict["key"]
```

## Lambda 進階用法

### 捕捉外部變數

```python
# 在 lambda 中使用外部變數
topics = ["a", "b", "c"]

# lambda 結合預設參數捕捉
self.client_.on_connect = lambda c, u, f, rc, p: self.onConnect(c, topics, rc)

# 等同於
def make_handler(captured_topics):
    def handler(c, u, f, rc, p):
        return self.onConnect(c, captured_topics, rc)
    return handler

self.client_.on_connect = make_handler(topics)
```

## 相關頁面

- [[entities/rclpy|rclpy]] - ROS2 Python 函式庫
- [[synthesis/ROS2-Python-模式|ROS2 Python 模式]] - ROS2 程式碼模式
- [[entities/MQTT|MQTT]] - MQTT 協定與 paho-mqtt
- [[synthesis/Omniverse-Extension-開發|Omniverse Extension 開發]] - 執行緒安全模式
