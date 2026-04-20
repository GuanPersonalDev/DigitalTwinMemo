---
title: factory-floor-digital-twin 專案
type: source
created: 2026-04-17
updated: 2026-04-18
original: https://github.com/GuanPersonalDev/factory-floor-digital-twin
tags: [專案, ROS2, 數位孿生, Python, MQTT, Docker, Omniverse]
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
│   └── factory_publisher.py              # 早期版本（未完成）
├── ros2_publisher/
│   └── machine_publisher.py              # ROS2 Publisher 節點
├── bridge/
│   ├── ros2_to_mqtt.py                   # ROS2 → MQTT 橋接器
│   └── config.py                         # 組態設定
├── omniverse_extension/                  # Omniverse Extension（新增）
│   ├── config/
│   │   └── extension.toml                # Extension 設定檔
│   └── omniverse_factory_twin/
│       ├── __init__.py                   # 模組初始化
│       └── extension.py                  # Extension 主程式
├── mosquitto/
│   └── config/mosquitto.conf             # MQTT Broker 設定
├── docker-compose.yml                    # Docker 服務定義
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

### ros2_to_mqtt.py

**功能**：訂閱 ROS2 topic，轉發到 MQTT broker

**架構**：
```
Ros2MqttBridge (Node)
├── mqttClient_: mqtt.Client        # MQTT 客戶端
├── subscriptions: Dict[str, Sub]   # 多個 ROS2 訂閱
├── connectMqtt()                   # 連線（含重試機制）
├── onRos2Message()                 # 收到訊息後發布到 MQTT
└── destroy_node()                  # 優雅關閉
```

**Topic 對應**：
| ROS2 Topic | MQTT Topic |
|------------|------------|
| `/factory/machine_01/status` | `factory/machine_01/status` |
| `/factory/machine_02/status` | `factory/machine_02/status` |
| `/factory/machine_03/status` | `factory/machine_03/status` |

**核心程式碼**：
```python
# 動態建立多個 subscription（使用 lambda 捕捉變數）
for ros2_topic, mqtt_topic in TOPIC_MAP.items():
    self.create_subscription(
        String,
        ros2_topic,
        lambda msg, t=mqtt_topic: self.onRos2Message(msg, t),
        10
    )

# 收到 ROS2 訊息後轉發到 MQTT（含錯誤處理）
def onRos2Message(self, msg, mqtt_topic):
    try:
        data = json.loads(msg.data)
        self.mqttClient_.publish(mqtt_topic, msg.data)
    except json.JSONDecodeError as e:
        self.get_logger().warn(f"JSON parse error: {e}")
    except Exception as e:
        self.get_logger().warn(f"Publish error: {e}")
```

**MQTT 連線重試機制**：
```python
def connectMqtt(self):
    retryCount = 0
    maxRetries = 5
    while retryCount < maxRetries:
        try:
            self.mqttClient_.connect(MQTT_BROKER_HOST, MQTT_BROKER_PORT)
            self.get_logger().info(f"Connected to MQTT Broker...")
            return
        except Exception as e:
            retryCount += 1
            self.get_logger().warn(f"connect fail ({retryCount}/{maxRetries}): {e}")
            time.sleep(2)
    raise RuntimeError("Connect to MQTT Broker fail")
```

**優雅關閉**：
```python
def destroy_node(self):
    self.mqttClient_.loop_stop()    # 停止背景迴圈
    self.mqttClient_.disconnect()   # 斷開連線
    super().destroy_node()          # 呼叫父類別

def main(args=None):
    rclpy.init(args=args)
    node = Ros2MqttBridge()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()
```

### config.py

組態設定模組：
```python
MQTT_BROKER_HOST = "10.255.255.254"
MQTT_BROKER_PORT = 1883

TOPIC_MAP = {
    "/factory/machine_01/status": "factory/machine_01/status",
    # ...
}
```

### Omniverse Extension（新增）

**功能**：在 Omniverse 中訂閱 MQTT 訊息，接收工廠機台資料

**架構**：
```
FactoryTwinExtension (omni.ext.IExt)
├── mqttClient_: mqtt.Client        # MQTT 客戶端
├── on_startup()                    # Extension 啟動
├── on_shutdown()                   # Extension 關閉
├── connectMqtt()                   # 連接 MQTT
├── onMqttConnect()                 # 連線成功回呼
└── onMqttMessage()                 # 收到訊息回呼
```

**extension.toml（Extension 設定）**：
```toml
[package]
title = "Factory Floor Digital Twin"
description = "Real-time factory floor monitoring via MQTT"
version = "0.1.0"

[[python.module]]
name = "omniverse_factory_twin"

[dependencies]
"omni.kit.uiapp" = {}
```

**extension.py 核心程式碼**：
```python
import omni.ext
import paho.mqtt.client as mqtt
import json

MQTT_BROKER_HOST = "localhost"
MQTT_BROKER_PORT = 1883
MACHINE_TOPICS = [
    "factory/machine_01/status",
    "factory/machine_02/status",
    "factory/machine_03/status",
]

class FactoryTwinExtension(omni.ext.IExt):

    def on_startup(self, ext_id):
        print("[Factory Twin] Extension activate")
        self.mqttClient_ = None
        self.connectMqtt()

    def on_shutdown(self):
        print("[Factory Twin] Extension end")
        if self.mqttClient_:
            self.mqttClient_.loop_stop()
            self.mqttClient_.disconnect()

    def connectMqtt(self):
        try:
            self.mqttClient_ = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
            self.mqttClient_.on_connect = self.onMqttConnect
            self.mqttClient_.on_message = self.onMqttMessage
            self.mqttClient_.connect(MQTT_BROKER_HOST, MQTT_BROKER_PORT)
            self.mqttClient_.loop_start()
        except Exception as e:
            print(f"[Factory Twin] MQTT connect error: {e}")

    def onMqttConnect(self, client, userdata, flags, reason_code, properties):
        if reason_code == 0:
            print("[Factory Twin] Connect to MQTT success")
            for topic in MACHINE_TOPICS:
                client.subscribe(topic)
        else:
            print(f"[Factory Twin] Connect fail: {reason_code}")

    def onMqttMessage(self, client, userdata, msg):
        try:
            data = json.loads(msg.payload.decode())
            print(f"[Factory Twin] {msg.topic} -> {data}")
            # TODO: 更新 3D 場景中的機台狀態
        except Exception as e:
            print(f"[Factory Twin] Message error: {e}")
```

### Docker 基礎設施

使用 Eclipse Mosquitto 作為 MQTT Broker：
```yaml
# docker-compose.yml
services:
  mosquitto:
    image: eclipse-mosquitto:2.0
    container_name: factory_mosquitto
    ports:
      - "1883:1883"
```

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
| `create_subscription()` | 建立訂閱者 |
| `destroy_node()` | 節點清理（可覆寫） |
| `rclpy.ok()` | 檢查 ROS2 是否仍在運行 |

### paho-mqtt

| API | 用途 |
|-----|------|
| `mqtt.Client()` | 建立 MQTT 客戶端 |
| `client.connect(host, port)` | 連接到 Broker |
| `client.loop_start()` | 啟動背景處理迴圈 |
| `client.publish(topic, msg)` | 發布訊息 |
| `client.loop_stop()` | 停止背景處理迴圈 |
| `client.disconnect()` | 斷開連線 |
| `client.on_connect` | 連線成功回呼（callback） |
| `client.on_message` | 收到訊息回呼（callback） |
| `client.subscribe(topic)` | 訂閱 topic |

### Omniverse Kit（新增）

| API | 用途 |
|-----|------|
| `omni.ext.IExt` | Extension 基底類別 |
| `on_startup(ext_id)` | Extension 啟動時呼叫 |
| `on_shutdown()` | Extension 關閉時呼叫 |

### Python 標準庫

| 模組 | 函式 | 用途 |
|------|------|------|
| `json` | `dumps()` | 字典轉 JSON 字串 |
| `random` | `uniform()` | 隨機浮點數 |
| `random` | `choice()` | 隨機選擇 |
| `time` | `sleep()` | 延遲執行 |

## Python 語法技巧

| 技巧 | 範例 |
|------|------|
| 字典推導式 | `{m: create_publisher(...) for m in MACHINES}` |
| f-string | `f"/factory/{machine}/status"` |
| `.items()` 遍歷 | `for k, v in dict.items()` |
| `round()` | `round(random.uniform(60, 95), 1)` |
| lambda 預設參數 | `lambda msg, t=topic: fn(msg, t)` |
| try/except/finally | 確保資源釋放 |
| while 重試迴圈 | 連線重試機制 |

## 執行方式

```bash
# 1. 啟動 MQTT Broker
docker-compose up -d

# 2. 執行 Publisher
cd ros2_publisher
python3 machine_publisher.py

# 3. 執行 Bridge（另一個終端）
cd bridge
python3 ros2_to_mqtt.py

# 監聽 ROS2 輸出
ros2 topic echo /factory/machine_01/status

# 監聽 MQTT 輸出
mosquitto_sub -h localhost -t "factory/#"
```

## 與數位孿生的關係

此專案是數位孿生系統的**資料層**，負責：
1. 模擬真實機台感測器
2. 透過 ROS2 topic 發布資料
3. 為 Omniverse 或其他視覺化平台提供資料來源

**整合架構（更新）**：
```
machine_publisher.py (ROS2 Publisher)
       ↓ ROS2 Topic
ros2_to_mqtt.py (Bridge)
       ↓ MQTT (Mosquitto Broker)
       ↓
┌──────┴──────────────────────┐
│                             │
│  FactoryTwinExtension       │ ← Omniverse Extension（新增）
│  (omniverse_factory_twin)   │
│                             │
└──────┬──────────────────────┘
       ↓
  Omniverse 3D 視覺化
```

## 已完成

- [x] ROS2 Publisher 節點
- [x] ROS2 → MQTT 橋接器
- [x] Docker 化 MQTT Broker
- [x] MQTT 連線重試機制
- [x] 優雅關閉（Graceful Shutdown）
- [x] Bridge 錯誤處理（JSON 解析、發布異常）
- [x] Omniverse Extension 基礎架構
- [x] Extension MQTT 訂閱機制

## 待改進

1. 增加更多感測器類型（電流、壓力等）
2. 加入異常狀態的模擬邏輯
3. ~~與 Omniverse 整合測試~~ ✓ Extension 已建立
4. ~~加入 Subscriber 節點接收控制指令~~ ✓ 已實作 Bridge
5. MQTT → ROS2 反向橋接（控制指令）
6. Extension 連接 3D 場景物件
7. 根據感測器資料更新機台視覺狀態

## 相關頁面

- [[rclpy]] - ROS2 Python 函式庫
- [[ROS2]] - 指令參考
- [[synthesis/ROS2-Python-模式|ROS2 Python 模式]] - 程式碼模式
- [[concepts/數位孿生|數位孿生]] - 概念說明
- [[entities/MQTT|MQTT]] - 訊息協定
- [[entities/NVIDIA Omniverse|NVIDIA Omniverse]] - Omniverse 平台
