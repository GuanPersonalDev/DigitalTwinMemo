---
title: factory-floor-digital-twin 專案
type: source
created: 2026-04-17
updated: 2026-04-29
original: https://github.com/GuanPersonalDev/factory-floor-digital-twin
tags: [專案, ROS2, 數位孿生, Python, MQTT, Docker, Omniverse, USD, TOML]
---

# factory-floor-digital-twin 專案

**作者**：GuanPersonalDev
**類型**：個人專案
**來源**：https://github.com/GuanPersonalDev/factory-floor-digital-twin

## 專案概述

工廠地板數位孿生專案，使用 ROS2 模擬工廠機台感測器資料發布，透過 MQTT 傳送到 Omniverse 進行 3D 視覺化。

## 檔案結構

```
factory-floor-digital-twin/
├── config/                               # 組態模組（新增）
│   ├── config_loader.py                  # 組態載入器
│   ├── machines.toml                     # 機台設定
│   ├── thresholds.toml                   # 閾值設定
│   └── topic_resolver.py                 # Topic 解析器
├── ros2_publisher/
│   ├── machine_publisher.py              # ROS2 Publisher 節點（重構）
│   ├── topic_data_generator.py           # 資料產生器（新增）
│   └── requirements.txt
├── bridge/
│   ├── ros2_to_mqtt.py                   # ROS2 → MQTT 橋接器（重構）
│   └── ros2_to_mqtt_config.py            # Bridge 組態
├── omniverse_extension/
│   ├── config/
│   │   └── extension.toml                # Extension 設定檔
│   └── omniverse_factory_twin/
│       ├── __init__.py                   # 模組初始化
│       ├── mqtt_client.py                # MQTT 客戶端封裝
│       ├── base_extension.py             # 抽象基底類別
│       ├── factory_log.py                # 日誌記錄系統（新增）
│       └── extension.py                  # Extension 主程式（重構）
├── external_assets/                      # 外部資產（新增）
│   └── USD_Explorer_Sample_NVD@10011/    # USD 場景資產
├── docs/
│   └── architecture.png                  # 架構圖
├── mosquitto/
│   └── config/mosquitto.conf             # MQTT Broker 設定
├── docker-compose.yml                    # Docker 服務定義
└── README.md
```

## 核心程式碼分析

### config 模組（新增）

專案在 04-25 引入集中式組態管理，將設定從程式碼中分離。

#### config_loader.py

**功能**：載入 TOML 組態檔，提供機台與閾值設定

**架構**：
```
MachineConfig（機台設定）
├── machine_id: str           # 機台 ID
├── display_name: str         # 顯示名稱
├── usd_prim_path: str        # USD 路徑
├── location: str             # 位置區域
├── getRos2Topic(param)       # 取得 ROS2 Topic
└── getMqttTopic(param)       # 取得 MQTT Topic

FactoryConfig（工廠設定）
├── machines: List[MachineConfig]   # 機台列表
├── getMachineById(id)              # 依 ID 取得機台
├── computeSeverity(param, value)   # 計算嚴重程度
├── resolveColor(mode, severity)    # 解析顏色
└── getOpacity(mode)                # 取得透明度
```

**嚴重程度定義**：
| 等級 | 說明 | 數值 |
|------|------|------|
| NORMAL | 正常 | 0 |
| WARNING | 警告 | 1 |
| ERROR | 錯誤 | 2 |

**操作模式**：
| 模式 | 透明度 | 說明 |
|------|--------|------|
| RUNNING | 1.0 | 運行中 |
| IDLE | 0.6 | 閒置 |
| SHUTDOWN | 0.4 | 關機 |
| OFFLINE | 0.3 | 離線 |

#### machines.toml

```toml
[[machines]]
machine_id = "machine_01"
display_name = "機器手臂 01"
usd_prim_path = "/World/Machine_01"
location = "Zone A"

[[machines]]
machine_id = "machine_04"
display_name = "機器手臂 04（狀態異常模擬）"
usd_prim_path = "/World/Machine_04"
location = "Zone D"
```

#### thresholds.toml

```toml
[parameters.temperature]
unit = "°C"
warning = 70.0
error = 85.0

[parameters.vibration]
unit = "mm/s"
warning = 5.0
error = 10.0

[operation_mode]
valid_modes = "RUNNING, IDLE, SHUTDOWN, OFFLINE"

[operation_mode.visual.SHUTDOWN]
opacity = 0.4
color = [0.4, 0.4, 0.4]

[severity_colors]
normal = [0.0, 0.8, 0.0]    # 綠色
warning = [1.0, 0.6, 0.0]   # 橘色
error = [1.0, 0.0, 0.0]     # 紅色
```

#### topic_resolver.py

**功能**：統一管理 Topic 命名規則

```python
_NAMESPACE = "factory"

def getRos2Topic(machine_id: str, param: str) -> str:
    return f"/{_NAMESPACE}/{machine_id}/{param}"
    # 例：/factory/machine_01/temperature

def getMqttTopic(machine_id: str, param: str) -> str:
    return f"{_NAMESPACE}/{machine_id}/{param}"
    # 例：factory/machine_01/temperature

def getMqttSubscribePattern(machine_id: str) -> str:
    return f"{_NAMESPACE}/{machine_id}/+"
    # 例：factory/machine_01/+（訂閱單機所有參數）

def getAllMachinesMqttPattern() -> str:
    return f"{_NAMESPACE}/#"
    # 例：factory/#（訂閱所有機台）

def parseMqttTopic(topic: str) -> tuple[str, str] | None:
    # "factory/machine_01/temperature" -> ("machine_01", "temperature")
    parts = topic.split("/")
    if len(parts) != 3 or parts[0] != _NAMESPACE:
        return None
    return parts[1], parts[2]
```

### topic_data_generator.py（新增）

**功能**：模擬機台狀態，支援腳本式狀態變化

**架構**：
```
ScriptPhase（腳本階段）
├── mode: str       # 操作模式
└── duration: int   # 持續時間（秒），None 為永久

MachineState（機台狀態）
├── machine_id: str
├── script: List[ScriptPhase]    # 狀態腳本
├── getCurrentMode()             # 取得當前模式
├── getParamValue(param)         # 取得參數值
└── getAllTopics()               # 取得所有 Topic 資料
```

**預設參數範圍**：
| 參數 | 最小值 | 最大值 |
|------|--------|--------|
| temperature | 60°C | 95°C |
| vibration | 0.1 mm/s | 5.0 mm/s |

**腳本式模擬範例**：
```python
# machine_04 使用預定義腳本
script = [
    ScriptPhase(mode="RUNNING", duration=5),   # 運行 5 秒
    ScriptPhase(mode="IDLE", duration=2),      # 閒置 2 秒
    ScriptPhase(mode="SHUTDOWN", duration=None) # 永久關機
]
state = MachineState("machine_04", script=script)
```

### machine_publisher.py（重構）

**功能**：模擬 4 台機台的感測器資料，每個參數獨立 Topic

**架構**：
```
MachinePublisher (Node)
├── factoryConfig_: FactoryConfig     # 組態
├── machineStates_: Dict[str, MachineState]  # 機台狀態
├── publishers_: Dict[str, Publisher]  # Publisher（每參數一個）
├── timer: Timer                       # 定時器（1秒）
└── publishMachineData()               # 發布資料
```

**Topic 結構（重構後）**：
```
/factory/machine_01/temperature      →  {"value": 75.5}
/factory/machine_01/vibration        →  {"value": 2.3}
/factory/machine_01/operation_mode   →  {"value": "RUNNING"}
/factory/machine_02/temperature      →  ...
...
/factory/machine_04/operation_mode   →  {"value": "SHUTDOWN"}
```

**參數定義**：
| 參數 | 類型 | 說明 |
|------|------|------|
| temperature | float | 溫度（°C） |
| vibration | float | 振動（mm/s） |
| operation_mode | string | 操作模式 |

### ros2_to_mqtt.py（重構）

**功能**：訂閱 ROS2 topic，轉發到 MQTT broker

**架構**：
```
Ros2MqttBridge (Node)
├── mqttClient_: mqtt.Client        # MQTT 客戶端
├── factoryConfig_: FactoryConfig   # 組態（新增）
├── subscriptions: Dict[str, Sub]   # 多個 ROS2 訂閱
├── connectMqtt()                   # 連線（含重試機制）
├── onRos2Message()                 # 收到訊息後發布到 MQTT
└── destroy_node()                  # 優雅關閉
```

**Topic 對應（重構後）**：
| ROS2 Topic | MQTT Topic |
|------------|------------|
| `/factory/machine_01/temperature` | `factory/machine_01/temperature` |
| `/factory/machine_01/vibration` | `factory/machine_01/vibration` |
| `/factory/machine_01/operation_mode` | `factory/machine_01/operation_mode` |
| ... | ... |

**核心程式碼**：
```python
# 使用 config 模組動態建立 subscription
from config.config_loader import FactoryConfig
from config.topic_resolver import getRos2Topic, getMqttTopic

PARAMS = ["temperature", "vibration", "operation_mode"]

for machine in self.factoryConfig_.machines:
    for param in PARAMS:
        ros2_topic = getRos2Topic(machine.machine_id, param)
        mqtt_topic = getMqttTopic(machine.machine_id, param)
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

### Omniverse Extension（重構）

**功能**：在 Omniverse 中訂閱 MQTT 訊息，接收工廠機台資料並更新 3D 場景

**架構（04-29 版本）**：
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
├── machineInfos_: Dict[str, MachineInfo]  # 機台資訊
├── factoryLog_: FactoryLog                # 日誌系統（新增）
├── onExtensionStartup()                   # 初始化
├── onMqttMessage()                        # 處理訊息、記錄日誌
├── onUpdate()                             # 主執行緒更新場景
└── updateMachineColor()                   # USD API 更新顏色

MachineInfo（機台資訊，新增）
├── machine_id: str
├── usd_path: str
└── calc_color(factoryLog, factoryConfig)  # 計算顏色

FactoryLog / MachineLog（日誌系統，新增）
├── record(machine_id, param, data)        # 記錄資料
├── getLatestMode(machine_id)              # 取得最新模式
└── getMachineLatestTopic(machine_id, param)  # 取得最新參數值
```

#### factory_log.py（日誌記錄系統，新增）

**功能**：記錄機台參數的歷史資料，支援查詢最新值

```python
from collections import deque
from datetime import datetime

class MachineLog:
    """單機日誌，使用固定大小循環緩衝區"""
    MAX_ENTRIES = 50

    def __init__(self):
        self.entries_ = deque(maxlen=self.MAX_ENTRIES)

    def append(self, param: str, data: dict):
        timestamp = datetime.now()
        self.entries_.append((timestamp, param, data))

    def getLatestByTopic(self, param: str):
        """從最新往回搜尋指定參數"""
        for entry in reversed(self.entries_):
            if entry[1] == param:
                return entry
        return None


class FactoryLog:
    """工廠日誌管理，管理多台機器"""

    def __init__(self):
        self.machines_: dict[str, MachineLog] = {}

    def record(self, machine_id: str, param: str, data: dict):
        if machine_id not in self.machines_:
            self.machines_[machine_id] = MachineLog()
        self.machines_[machine_id].append(param, data)

    def getLatestMode(self, machine_id: str) -> str | None:
        """取得機台最新操作模式"""
        entry = self.getMachineLatestTopic(machine_id, "operation_mode")
        if entry:
            return entry[2].get("value")
        return None

    def getMachineLatestTopic(self, machine_id: str, param: str):
        if machine_id not in self.machines_:
            return None
        return self.machines_[machine_id].getLatestByTopic(param)
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

#### extension.py（實作類別，04-29 版本）

```python
import omni.kit.app
from pxr import Gf, UsdGeom
import omni.usd
import threading
from .base_extension import BaseMqttExtension
from .factory_log import FactoryLog
from config.config_loader import FactoryConfig
from config.topic_resolver import getAllMachinesMqttPattern, parseMqttTopic

PARAMS = ["temperature", "vibration", "operation_mode"]


class MachineInfo:
    """機台資訊，負責計算顏色與透明度"""

    def __init__(self, machine_id: str, usd_path: str):
        self.machine_id_ = machine_id
        self.usd_path_ = usd_path

    def calc_color(self, factoryLog: FactoryLog, factoryConfig: FactoryConfig):
        """根據日誌中的最新資料計算顏色"""
        # 取得最新操作模式
        mode = factoryLog.getLatestMode(self.machine_id_) or "RUNNING"

        # 計算最高嚴重程度
        max_severity = "NORMAL"
        max_rank = 0

        for param in ["temperature", "vibration"]:
            entry = factoryLog.getMachineLatestTopic(self.machine_id_, param)
            if entry:
                value = entry[2].get("value")
                if value is not None:
                    severity, rank = factoryConfig.computeSeverity(param, value)
                    if rank > max_rank:
                        max_severity = severity
                        max_rank = rank

        # 根據模式和嚴重程度解析顏色
        color = factoryConfig.resolveColor(mode, max_severity)
        opacity = factoryConfig.getOpacity(mode)
        return color, opacity


class FactoryTwinExtension(BaseMqttExtension):

    MQTT_HOST = "localhost"
    MQTT_PORT = 1883

    def getMqttTopics(self) -> list[str]:
        return [getAllMachinesMqttPattern()]  # "factory/#"

    def onExtensionStartup(self, ext_id):
        self.factoryConfig_ = FactoryConfig()
        self.factoryLog_ = FactoryLog()
        self.lock_ = threading.Lock()

        # 建立 MachineInfo
        self.machineInfos_ = {
            m.machine_id: MachineInfo(m.machine_id, m.usd_prim_path)
            for m in self.factoryConfig_.machines
        }

        # 訂閱 Omniverse 更新事件
        self.updateSub_ = omni.kit.app.get_app().get_update_event_stream()\
            .create_subscription_to_pop(self.onUpdate, name="factory_twin_update")

    def onUpdate(self, event):
        """主執行緒：計算每台機台的顏色並更新"""
        with self.lock_:
            for machine_id, info in self.machineInfos_.items():
                color, opacity = info.calc_color(
                    self.factoryLog_, self.factoryConfig_
                )
                self.updateMachineColor(info.usd_path_, color, opacity)

    def onMqttMessage(self, topic: str, data: dict):
        """MQTT 執行緒：解析 Topic 並記錄資料"""
        result = parseMqttTopic(topic)
        if not result:
            return
        machine_id, param = result
        with self.lock_:
            self.factoryLog_.record(machine_id, param, data)

    def updateMachineColor(self, usd_path: str, color: list, opacity: float):
        try:
            stage = omni.usd.get_context().get_stage()
            prim = stage.GetPrimAtPath(usd_path)
            if not prim.IsValid():
                return
            gprim = UsdGeom.Gprim(prim)
            gprim.GetDisplayColorAttr().Set([Gf.Vec3f(*color)])
            gprim.GetDisplayOpacityAttr().Set([opacity])
        except Exception as e:
            print(f"[Factory Twin] Update error: {usd_path} -> {e}")
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

### USD / pxr

| API | 用途 |
|-----|------|
| `omni.usd.get_context().get_stage()` | 取得目前的 USD Stage |
| `stage.GetPrimAtPath(path)` | 取得指定路徑的 Prim |
| `prim.IsValid()` | 檢查 Prim 是否存在 |
| `UsdGeom.Gprim(prim)` | 將 Prim 轉為幾何物件 |
| `GetDisplayColorAttr().Set()` | 設定顯示顏色 |
| `GetDisplayOpacityAttr().Set()` | 設定顯示透明度（新增） |
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
| `collections` | `deque(maxlen=N)` | 固定大小佇列（新增） |
| `datetime` | `datetime.now()` | 取得當前時間（新增） |
| `dataclasses` | `@dataclass` | 資料類別（新增） |

### TOML 解析（新增）

| 模組 | 版本 | 用途 |
|------|------|------|
| `tomllib` | Python 3.11+ | 內建 TOML 解析 |
| `tomli` | Python < 3.11 | 第三方 TOML 解析 |

```python
# 相容寫法
try:
    import tomllib
except ImportError:
    import tomli as tomllib

with open("config.toml", "rb") as f:
    config = tomllib.load(f)
```

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

**整合架構（v3）**：
```
┌─────────────────────────────────────────────────────────────┐
│  config/                                                     │
│  ├── machines.toml     （機台設定）                          │
│  ├── thresholds.toml   （閾值設定）                          │
│  ├── config_loader.py  （FactoryConfig、MachineConfig）      │
│  └── topic_resolver.py （Topic 命名規則）                    │
└─────────────────────────────────────────────────────────────┘
                    ↓ 匯入
┌─────────────────────────────────────────────────────────────┐
│  ros2_publisher/                                             │
│  ├── topic_data_generator.py  (MachineState、ScriptPhase)   │
│  └── machine_publisher.py     (ROS2 Publisher)               │
│                                                              │
│  發布 Topics：                                               │
│  /factory/machine_01/temperature                             │
│  /factory/machine_01/vibration                               │
│  /factory/machine_01/operation_mode                          │
│  ...（4 台機台 × 3 個參數 = 12 個 Topic）                    │
└─────────────────────────────────────────────────────────────┘
                    ↓ ROS2 Topic
┌─────────────────────────────────────────────────────────────┐
│  bridge/ros2_to_mqtt.py                                      │
│  （訂閱 ROS2 Topic → 轉發 MQTT）                             │
└─────────────────────────────────────────────────────────────┘
                    ↓ MQTT (Mosquitto Broker)
┌─────────────────────────────────────────────────────────────┐
│  omniverse_extension/                                        │
│                                                              │
│  MqttClient → BaseMqttExtension                              │
│                    ↑ 繼承                                    │
│            FactoryTwinExtension                              │
│                    │                                         │
│            ┌───────┴───────┐                                 │
│            ↓               ↓                                 │
│      FactoryLog      MachineInfo                             │
│    （日誌記錄）    （顏色計算）                              │
│                            ↓                                 │
│                    USD Scene Update                          │
│            (顏色 + 透明度根據嚴重程度變化)                   │
└─────────────────────────────────────────────────────────────┘
                    ↓
           Omniverse 3D 視覺化
     （4 台機台即時顏色/透明度反映狀態）
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
- [x] **組態模組化**（TOML 設定檔分離）
- [x] **Topic 分割**（每參數獨立 Topic）
- [x] **腳本式狀態模擬**（ScriptPhase、MachineState）
- [x] **新增 machine_04**（狀態異常模擬機台）
- [x] **日誌記錄系統**（FactoryLog、MachineLog）
- [x] **嚴重程度計算**（NORMAL/WARNING/ERROR）
- [x] **透明度控制**（根據操作模式）
- [x] **外部 USD 資產整合**

## 待改進

1. ~~增加更多感測器類型~~ ✓ 已支援 temperature/vibration/operation_mode
2. ~~加入異常狀態的模擬邏輯~~ ✓ machine_04 使用 ScriptPhase
3. ~~與 Omniverse 整合測試~~ ✓ Extension 已建立
4. ~~加入 Subscriber 節點接收控制指令~~ ✓ 已實作 Bridge
5. MQTT → ROS2 反向橋接（控制指令）
6. ~~Extension 連接 3D 場景物件~~ ✓ 已實作 USD 更新
7. ~~根據感測器資料更新機台視覺狀態~~ ✓ 已實作顏色+透明度
8. ~~根據溫度/振動值漸變顏色~~ ✓ 已實作嚴重程度計算
9. 加入 UI 面板顯示即時數據
10. 實作告警視覺效果（閃爍、發光）
11. 增加更多感測器（電流、壓力、轉速）
12. 持久化日誌（寫入檔案或資料庫）

## 相關頁面

- [[rclpy]] - ROS2 Python 函式庫
- [[ROS2]] - 指令參考
- [[synthesis/ROS2-Python-模式|ROS2 Python 模式]] - 程式碼模式
- [[concepts/數位孿生|數位孿生]] - 概念說明
- [[entities/MQTT|MQTT]] - 訊息協定
- [[entities/NVIDIA Omniverse|NVIDIA Omniverse]] - Omniverse 平台
