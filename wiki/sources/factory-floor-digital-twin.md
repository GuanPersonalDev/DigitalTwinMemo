---
title: factory-floor-digital-twin 專案
type: source
created: 2026-04-17
updated: 2026-04-22
original: https://github.com/GuanPersonalDev/factory-floor-digital-twin
tags: [專案, ROS2, 數位孿生, Python, MQTT, Docker, Omniverse, USD]
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
├── omniverse_extension/                  # Omniverse Extension
│   ├── config/
│   │   └── extension.toml                # Extension 設定檔
│   └── omniverse_factory_twin/
│       ├── __init__.py                   # 模組初始化
│       ├── mqtt_client.py                # MQTT 客戶端封裝（新增）
│       ├── base_extension.py             # 抽象基底類別（新增）
│       └── extension.py                  # Extension 主程式（重構）
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

### Omniverse Extension（重構）

**功能**：在 Omniverse 中訂閱 MQTT 訊息，接收工廠機台資料並更新 3D 場景

**架構（重構後）**：
```
MqttClient（MQTT 通訊封裝）
├── connect() / disconnect()
├── onConnect() / onMessage()
└── setMessageCallback()
       ↑ 使用
BaseMqttExtension（抽象基底類別）
├── on_startup() / on_shutdown()
├── onExtensionStartup()        # Hook 給子類別
├── onExtensionShutdown()       # Hook 給子類別
└── onMqttMessage()             # 抽象方法
       ↑ 繼承
FactoryTwinExtension（實作類別）
├── onExtensionStartup()        # 初始化執行緒鎖
├── onMqttMessage()             # 處理訊息、排隊更新
├── onUpdate()                  # 主執行緒更新場景
├── statusToColor()             # 狀態→顏色對應
└── updateMachineColor()        # USD API 更新顏色
```

#### mqtt_client.py（MQTT 客戶端封裝）

```python
import paho.mqtt.client as mqtt
import json
from typing import Callable, Optional

class MqttClient:
    def __init__(self, host: str, port: int):
        self.host_ = host
        self.port_ = port
        self.client_ = None
        self.messageCallback_: Optional[Callable] = None

    def setMessageCallback(self, callback: Callable):
        self.messageCallback_ = callback

    def connect(self, topics: list[str]):
        try:
            self.client_ = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
            self.client_.on_connect = lambda c, u, f, rc, p: self.onConnect(c, topics, rc)
            self.client_.on_message = self.onMessage
            self.client_.connect(self.host_, self.port_)
            self.client_.loop_start()
        except Exception as e:
            print(f"[Mqtt Client] Connect error: {e}")

    def disconnect(self):
        if self.client_:
            self.client_.loop_stop()
            self.client_.disconnect()

    def onConnect(self, client, topics: list[str], reason_code):
        if reason_code == 0:
            print(f"[Mqtt Client] Connect success: {self.host_}:{self.port_}")
            for topic in topics:
                client.subscribe(topic)
        else:
            print(f"[Mqtt Client] Connect fail: {reason_code}")

    def onMessage(self, client, userdata, msg):
        try:
            data = json.loads(msg.payload.decode())
            if self.messageCallback_:
                self.messageCallback_(msg.topic, data)
        except json.JSONDecodeError as e:
            print(f"[Mqtt Client] Json parse error: {e}")
```

#### base_extension.py（抽象基底類別）

```python
import omni.ext
from .mqtt_client import MqttClient

class BaseMqttExtension(omni.ext.IExt):

    MQTT_HOST = "localhost"
    MQTT_PORT = 1883
    MQTT_TOPICS: list[str] = []

    def on_startup(self, ext_id):
        print(f"[{self.__class__.__name__}] activated")
        self.mqttClient_ = MqttClient(self.MQTT_HOST, self.MQTT_PORT)
        self.mqttClient_.setMessageCallback(self.onMqttMessage)
        self.onExtensionStartup(ext_id)
        self.mqttClient_.connect(self.MQTT_TOPICS)

    def on_shutdown(self):
        print(f"[{self.__class__.__name__}] shutdown")
        self.onExtensionShutdown()
        if hasattr(self, 'mqttClient_') and self.mqttClient_:
            self.mqttClient_.disconnect()
            self.mqttClient_ = None

    def onExtensionStartup(self, ext_id):
        pass  # Hook 給子類別覆寫

    def onExtensionShutdown(self):
        pass  # Hook 給子類別覆寫

    def onMqttMessage(self, topic: str, data: dict):
        raise NotImplementedError  # 子類別必須實作
```

#### extension.py（實作類別）

```python
import omni.kit.app
from pxr import Gf, UsdGeom
import omni.usd
import threading
from .base_extension import BaseMqttExtension

MACHINE_USD_PATHS = {
    "factory/machine_01/status": "/World/Machine_01",
    "factory/machine_02/status": "/World/Machine_02",
    "factory/machine_03/status": "/World/Machine_03",
}

class FactoryTwinExtension(BaseMqttExtension):

    MQTT_BROKER_HOST = "localhost"
    MQTT_BROKER_PORT = 1883
    MQTT_TOPICS = list(MACHINE_USD_PATHS.keys())

    def onExtensionStartup(self, ext_id):
        self.pendingUpdates_ = {}
        self.lock_ = threading.Lock()
        # 訂閱 Omniverse 更新事件（主執行緒）
        self.updateSub_ = omni.kit.app.get_app().get_update_event_stream()\
            .create_subscription_to_pop(
            self.onUpdate, name="factory_twin_update"
        )

    def onUpdate(self, event):
        # 在主執行緒中安全更新 USD 場景
        with self.lock_:
            updates = dict(self.pendingUpdates_)  # 複製字典
            self.pendingUpdates_.clear()
        for usd_path, color in updates.items():
            self.updateMachineColor(usd_path, color)

    def onExtensionShutdown(self):
        self.updateSub_ = None

    def onMqttMessage(self, topic: str, data: dict):
        # MQTT 執行緒中執行
        usd_path = MACHINE_USD_PATHS.get(topic)
        if not usd_path:
            return
        status = data.get("status", "unknown")
        color = self.statusToColor(status)
        with self.lock_:
            self.pendingUpdates_[usd_path] = color

    def statusToColor(self, status: str) -> Gf.Vec3f:
        colorMap = {
            "running": Gf.Vec3f(0.0, 1.0, 0.0),  # 綠色
            "warning": Gf.Vec3f(1.0, 1.0, 0.0),  # 黃色
            "error": Gf.Vec3f(1.0, 0.0, 0.0),    # 紅色
        }
        return colorMap.get(status, Gf.Vec3f(0.5, 0.5, 0.5))  # 預設灰色

    def updateMachineColor(self, usd_path: str, color: Gf.Vec3f):
        try:
            stage = omni.usd.get_context().get_stage()
            prim = stage.GetPrimAtPath(usd_path)
            if not prim.IsValid():
                print(f"[Factory Twin] Not found prim: {usd_path}")
                return
            UsdGeom.Gprim(prim).GetDisplayColorAttr().Set(
                [(color[0], color[1], color[2])]
            )
        except Exception as e:
            print(f"[Factory Twin] Update color error: {usd_path} -> {e}")
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

### Omniverse Kit

| API | 用途 |
|-----|------|
| `omni.ext.IExt` | Extension 基底類別 |
| `on_startup(ext_id)` | Extension 啟動時呼叫 |
| `on_shutdown()` | Extension 關閉時呼叫 |
| `omni.kit.app.get_app()` | 取得應用程式實例 |
| `get_update_event_stream()` | 取得更新事件串流 |
| `create_subscription_to_pop()` | 訂閱更新事件（每幀呼叫） |

### USD / pxr（新增）

| API | 用途 |
|-----|------|
| `omni.usd.get_context().get_stage()` | 取得目前的 USD Stage |
| `stage.GetPrimAtPath(path)` | 取得指定路徑的 Prim |
| `prim.IsValid()` | 檢查 Prim 是否存在 |
| `UsdGeom.Gprim(prim)` | 將 Prim 轉為幾何物件 |
| `GetDisplayColorAttr().Set()` | 設定顯示顏色 |
| `Gf.Vec3f(r, g, b)` | 3D 向量（RGB 顏色） |

### Python 標準庫

| 模組 | 函式 | 用途 |
|------|------|------|
| `json` | `dumps()` | 字典轉 JSON 字串 |
| `json` | `loads()` | JSON 字串轉字典 |
| `random` | `uniform()` | 隨機浮點數 |
| `random` | `choice()` | 隨機選擇 |
| `time` | `sleep()` | 延遲執行 |
| `threading` | `Lock()` | 執行緒鎖 |
| `typing` | `Callable`, `Optional` | 類型提示 |

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
| `list[str]` | Python 3.9+ 泛型類型提示 |
| `Optional[Callable]` | 可選回呼類型 |
| `hasattr(obj, 'attr')` | 安全屬性檢查 |
| `raise NotImplementedError` | 抽象方法模式 |
| `dict(d)` | 字典淺複製 |
| `with lock:` | 執行緒安全區塊 |
| `self.__class__.__name__` | 取得類別名稱 |

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

**整合架構（v2）**：
```
machine_publisher.py (ROS2 Publisher)
       ↓ ROS2 Topic
ros2_to_mqtt.py (Bridge)
       ↓ MQTT (Mosquitto Broker)
       ↓
┌──────┴──────────────────────────────────────┐
│  Omniverse Extension                         │
│                                              │
│  MqttClient ──→ BaseMqttExtension            │
│                      ↑ 繼承                  │
│               FactoryTwinExtension           │
│                      ↓                       │
│              USD Scene Update                │
│       (Machine_01/02/03 顏色變化)            │
└──────────────────────────────────────────────┘
       ↓
  Omniverse 3D 視覺化（即時顏色反映狀態）
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
- [x] MQTT 客戶端封裝（MqttClient）
- [x] 抽象基底類別（BaseMqttExtension）
- [x] 執行緒安全場景更新（Lock + pendingUpdates）
- [x] USD 場景顏色更新（根據 status 變色）

## 待改進

1. 增加更多感測器類型（電流、壓力等）
2. 加入異常狀態的模擬邏輯
3. ~~與 Omniverse 整合測試~~ ✓ Extension 已建立
4. ~~加入 Subscriber 節點接收控制指令~~ ✓ 已實作 Bridge
5. MQTT → ROS2 反向橋接（控制指令）
6. ~~Extension 連接 3D 場景物件~~ ✓ 已實作 USD 更新
7. ~~根據感測器資料更新機台視覺狀態~~ ✓ 已實作顏色變化
8. 根據溫度/振動值漸變顏色
9. 加入 UI 面板顯示即時數據
10. 實作告警視覺效果（閃爍、發光）

## 相關頁面

- [[rclpy]] - ROS2 Python 函式庫
- [[ROS2]] - 指令參考
- [[synthesis/ROS2-Python-模式|ROS2 Python 模式]] - 程式碼模式
- [[concepts/數位孿生|數位孿生]] - 概念說明
- [[entities/MQTT|MQTT]] - 訊息協定
- [[entities/NVIDIA Omniverse|NVIDIA Omniverse]] - Omniverse 平台
