---
title: Omniverse Extension 開發
type: synthesis
created: 2026-04-19
updated: 2026-04-29
sources: [factory-floor-digital-twin]
tags: [Omniverse, Extension, Python, 開發, USD, 執行緒, 日誌]
---

# Omniverse Extension 開發

NVIDIA Omniverse 的 Extension 開發模式與實作範例。

## Extension 概念

Extension 是 Omniverse 的模組化擴充系統，可以：
- 新增 UI 元件
- 擴充功能
- 整合外部系統（如 MQTT）
- 自訂工作流程

## 檔案結構

```
omniverse_extension/
├── config/
│   └── extension.toml      # Extension 設定檔
└── omniverse_factory_twin/ # Python 模組
    ├── __init__.py         # 模組初始化
    ├── mqtt_client.py      # MQTT 客戶端封裝
    ├── base_extension.py   # 抽象基底類別
    └── extension.py        # 主程式（繼承基底）
```

## extension.toml 設定

Extension 的基本設定檔：

```toml
[package]
title = "Factory Floor Digital Twin"          # 顯示名稱
description = "Real-time factory monitoring"  # 描述
version = "0.1.0"                             # 版本號

[[python.module]]
name = "omniverse_factory_twin"               # Python 模組名稱

[dependencies]
"omni.kit.uiapp" = {}                         # 依賴的 Extension
```

### 常用設定項

| 設定 | 說明 |
|------|------|
| `[package]` | 基本資訊 |
| `[[python.module]]` | Python 模組定義 |
| `[dependencies]` | 依賴的其他 Extension |
| `[[test]]` | 測試設定 |

## Extension 類別結構

### 基本模板

```python
import omni.ext

class MyExtension(omni.ext.IExt):

    def on_startup(self, ext_id):
        """Extension 啟動時呼叫"""
        print(f"[MyExtension] Startup: {ext_id}")
        # 初始化程式碼

    def on_shutdown(self):
        """Extension 關閉時呼叫"""
        print("[MyExtension] Shutdown")
        # 清理程式碼
```

### 生命週期

```
Extension 載入
      ↓
on_startup(ext_id) 呼叫
      ↓
Extension 運行中
      ↓
on_shutdown() 呼叫
      ↓
Extension 卸載
```

## 專案實例：FactoryTwinExtension

### 架構圖

```
MqttClient（通訊封裝）
    ↑ 使用
BaseMqttExtension（抽象基底）
    ↑ 繼承
FactoryTwinExtension（實作）
    → USD Scene 更新
```

### mqtt_client.py（MQTT 封裝）

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
        self.client_ = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
        # lambda 捕捉 topics 變數
        self.client_.on_connect = lambda c, u, f, rc, p: self.onConnect(c, topics, rc)
        self.client_.on_message = self.onMessage
        self.client_.connect(self.host_, self.port_)
        self.client_.loop_start()

    def disconnect(self):
        if self.client_:
            self.client_.loop_stop()
            self.client_.disconnect()

    def onConnect(self, client, topics: list[str], reason_code):
        if reason_code == 0:
            for topic in topics:
                client.subscribe(topic)

    def onMessage(self, client, userdata, msg):
        data = json.loads(msg.payload.decode())
        if self.messageCallback_:
            self.messageCallback_(msg.topic, data)
```

### base_extension.py（抽象基底）

```python
import omni.ext
from .mqtt_client import MqttClient

class BaseMqttExtension(omni.ext.IExt):

    MQTT_HOST = "localhost"
    MQTT_PORT = 1883
    MQTT_TOPICS: list[str] = []  # 子類別覆寫

    def on_startup(self, ext_id):
        print(f"[{self.__class__.__name__}] activated")
        self.mqttClient_ = MqttClient(self.MQTT_HOST, self.MQTT_PORT)
        self.mqttClient_.setMessageCallback(self.onMqttMessage)
        self.onExtensionStartup(ext_id)  # Hook
        self.mqttClient_.connect(self.MQTT_TOPICS)

    def on_shutdown(self):
        self.onExtensionShutdown()  # Hook
        if hasattr(self, 'mqttClient_') and self.mqttClient_:
            self.mqttClient_.disconnect()

    def onExtensionStartup(self, ext_id):
        pass  # 子類別覆寫

    def onExtensionShutdown(self):
        pass  # 子類別覆寫

    def onMqttMessage(self, topic: str, data: dict):
        raise NotImplementedError  # 子類別必須實作
```

### extension.py（實作）

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

    MQTT_TOPICS = list(MACHINE_USD_PATHS.keys())

    def onExtensionStartup(self, ext_id):
        self.pendingUpdates_ = {}
        self.lock_ = threading.Lock()
        # 訂閱主執行緒更新事件
        self.updateSub_ = omni.kit.app.get_app().get_update_event_stream()\
            .create_subscription_to_pop(self.onUpdate, name="factory_twin_update")

    def onUpdate(self, event):
        with self.lock_:
            updates = dict(self.pendingUpdates_)
            self.pendingUpdates_.clear()
        for usd_path, color in updates.items():
            self.updateMachineColor(usd_path, color)

    def onMqttMessage(self, topic: str, data: dict):
        usd_path = MACHINE_USD_PATHS.get(topic)
        if not usd_path:
            return
        status = data.get("status", "unknown")
        color = self.statusToColor(status)
        with self.lock_:
            self.pendingUpdates_[usd_path] = color

    def statusToColor(self, status: str) -> Gf.Vec3f:
        colorMap = {
            "running": Gf.Vec3f(0.0, 1.0, 0.0),
            "warning": Gf.Vec3f(1.0, 1.0, 0.0),
            "error": Gf.Vec3f(1.0, 0.0, 0.0),
        }
        return colorMap.get(status, Gf.Vec3f(0.5, 0.5, 0.5))

    def updateMachineColor(self, usd_path: str, color: Gf.Vec3f):
        stage = omni.usd.get_context().get_stage()
        prim = stage.GetPrimAtPath(usd_path)
        if not prim.IsValid():
            return
        UsdGeom.Gprim(prim).GetDisplayColorAttr().Set(
            [(color[0], color[1], color[2])]
        )
```

## paho-mqtt 回呼函式

### on_connect 簽名

```python
def on_connect(client, userdata, flags, reason_code, properties):
    """
    client: mqtt.Client 實例
    userdata: 使用者資料
    flags: 連線旗標
    reason_code: 0 表示成功，其他為錯誤碼
    properties: MQTT 5.0 屬性
    """
    if reason_code == 0:
        # 連線成功
        client.subscribe("topic")
```

### on_message 簽名

```python
def on_message(client, userdata, msg):
    """
    client: mqtt.Client 實例
    userdata: 使用者資料
    msg: 訊息物件
        - msg.topic: topic 名稱
        - msg.payload: 訊息內容（bytes）
    """
    data = json.loads(msg.payload.decode())
```

## 常用 Omniverse API

### 更新事件訂閱

MQTT 訊息在背景執行緒中接收，但 USD 場景操作必須在主執行緒中執行。使用更新事件訂閱解決：

```python
import omni.kit.app

# 訂閱每幀更新事件
self.updateSub_ = omni.kit.app.get_app().get_update_event_stream()\
    .create_subscription_to_pop(
        self.onUpdate,           # 回呼函式
        name="my_update_handler" # 識別名稱
    )

def onUpdate(self, event):
    # 此處在主執行緒中執行
    pass

# 清理時取消訂閱
self.updateSub_ = None
```

### 執行緒安全模式

```python
import threading

class MyExtension:
    def __init__(self):
        self.pendingUpdates_ = {}     # 待處理的更新
        self.lock_ = threading.Lock()  # 執行緒鎖

    def onMqttMessage(self, topic, data):
        # 背景執行緒中
        with self.lock_:
            self.pendingUpdates_[topic] = data

    def onUpdate(self, event):
        # 主執行緒中
        with self.lock_:
            updates = dict(self.pendingUpdates_)  # 複製
            self.pendingUpdates_.clear()
        for topic, data in updates.items():
            self.processUpdate(topic, data)  # 安全更新 USD
```

### 場景操作

```python
from pxr import Gf, UsdGeom
import omni.usd

# 取得目前 stage
stage = omni.usd.get_context().get_stage()

# 取得 prim（場景物件）
prim = stage.GetPrimAtPath("/World/Machine_01")

# 檢查 prim 是否存在
if not prim.IsValid():
    print("Prim not found")
    return

# 設定顯示顏色
color = Gf.Vec3f(1.0, 0.0, 0.0)  # RGB 紅色
UsdGeom.Gprim(prim).GetDisplayColorAttr().Set(
    [(color[0], color[1], color[2])]
)

# 修改位移
prim.GetAttribute("xformOp:translate").Set((0, 0, 0))
```

### USD API 快速參考

| API | 用途 |
|-----|------|
| `omni.usd.get_context().get_stage()` | 取得 Stage |
| `stage.GetPrimAtPath(path)` | 取得 Prim |
| `prim.IsValid()` | 檢查存在 |
| `UsdGeom.Gprim(prim)` | 轉為幾何物件 |
| `GetDisplayColorAttr().Set()` | 設定顏色 |
| `Gf.Vec3f(r, g, b)` | 顏色向量 |

### UI 元件（未來擴充）

```python
import omni.ui as ui

# 建立視窗
window = ui.Window("Factory Twin", width=300, height=200)

with window.frame:
    with ui.VStack():
        ui.Label("Machine Status")
        ui.Button("Connect MQTT")
```

## 安裝與測試

### 開發模式載入

1. 開啟 Omniverse 應用程式
2. Window → Extensions
3. 點擊齒輪圖示 → Extension Search Paths
4. 加入你的 Extension 路徑
5. 搜尋並啟用 Extension

### 目錄結構要求

```
extension_root/
├── config/
│   └── extension.toml    # 必須存在
└── module_name/
    └── __init__.py       # 必須存在
```

## 日誌記錄系統（04-29 新增）

### 架構

```
FactoryLog（工廠日誌）
├── machines_: Dict[str, MachineLog]
├── record(machine_id, param, data)
├── getLatestMode(machine_id)
└── getMachineLatestTopic(machine_id, param)

MachineLog（單機日誌）
├── entries_: deque(maxlen=50)  # 固定大小循環緩衝區
├── append(param, data)
└── getLatestByTopic(param)
```

### 實作

```python
from collections import deque
from datetime import datetime

class MachineLog:
    MAX_ENTRIES = 50

    def __init__(self):
        self.entries_ = deque(maxlen=self.MAX_ENTRIES)

    def append(self, param: str, data: dict):
        self.entries_.append((datetime.now(), param, data))

    def getLatestByTopic(self, param: str):
        for entry in reversed(self.entries_):
            if entry[1] == param:
                return entry
        return None

class FactoryLog:
    def __init__(self):
        self.machines_: dict[str, MachineLog] = {}

    def record(self, machine_id: str, param: str, data: dict):
        if machine_id not in self.machines_:
            self.machines_[machine_id] = MachineLog()
        self.machines_[machine_id].append(param, data)
```

### 整合到 Extension

```python
def onMqttMessage(self, topic: str, data: dict):
    result = parseMqttTopic(topic)
    if not result:
        return
    machine_id, param = result
    with self.lock_:
        self.factoryLog_.record(machine_id, param, data)

def onUpdate(self, event):
    for machine_id, info in self.machineInfos_.items():
        # 從日誌計算顏色
        color, opacity = info.calc_color(
            self.factoryLog_, self.factoryConfig_
        )
        self.updateMachineColor(info.usd_path_, color, opacity)
```

## 嚴重程度與視覺反饋（04-29 新增）

### 嚴重程度計算

```python
def computeSeverity(self, param: str, value: float) -> tuple[str, int]:
    """
    根據參數值計算嚴重程度
    回傳 (severity_name, rank)
    """
    thresholds = self.thresholds_[param]
    if value >= thresholds["error"]:
        return ("ERROR", 2)
    elif value >= thresholds["warning"]:
        return ("WARNING", 1)
    else:
        return ("NORMAL", 0)
```

### 顏色解析

```python
def resolveColor(self, mode: str, severity: str) -> list[float]:
    """根據操作模式和嚴重程度解析顏色"""
    if mode in ["SHUTDOWN", "OFFLINE"]:
        # 關機/離線使用灰色
        return self.modeColors_[mode]
    # 運行中根據嚴重程度
    return self.severityColors_[severity]
```

### 透明度控制

```python
def getOpacity(self, mode: str) -> float:
    """根據操作模式取得透明度"""
    return {
        "RUNNING": 1.0,
        "IDLE": 0.6,
        "SHUTDOWN": 0.4,
        "OFFLINE": 0.3
    }.get(mode, 1.0)
```

### 更新 USD 顏色與透明度

```python
def updateMachineColor(self, usd_path: str, color: list, opacity: float):
    stage = omni.usd.get_context().get_stage()
    prim = stage.GetPrimAtPath(usd_path)
    if not prim.IsValid():
        return
    gprim = UsdGeom.Gprim(prim)
    gprim.GetDisplayColorAttr().Set([Gf.Vec3f(*color)])
    gprim.GetDisplayOpacityAttr().Set([opacity])  # 新增透明度
```

## 下一步

- [x] 連接 3D 場景物件 ✓
- [x] 根據狀態變更顏色 ✓
- [x] 根據溫度/振動值漸變顏色 ✓（嚴重程度計算）
- [x] 透明度控制 ✓
- [ ] 根據振動播放動畫
- [ ] 加入 UI 面板顯示即時數據
- [ ] 實作告警視覺效果（閃爍、發光）
- [ ] 持久化日誌

## 設計模式摘要

| 模式 | 用途 | 範例 |
|------|------|------|
| 抽象基底類別 | 共用 MQTT 連線邏輯 | `BaseMqttExtension` |
| Hook 方法 | 讓子類別擴充行為 | `onExtensionStartup()` |
| 回呼注入 | 解耦訊息處理 | `setMessageCallback()` |
| 執行緒安全更新 | 跨執行緒資料傳遞 | `Lock + pendingUpdates` |
| 狀態對應 | 資料→視覺轉換 | `statusToColor()` |
| 循環緩衝區 | 固定大小日誌 | `deque(maxlen=50)` |
| 組態分離 | TOML 設定檔 | `FactoryConfig` |

## 相關頁面

- [[entities/NVIDIA Omniverse|NVIDIA Omniverse]] - 平台介紹
- [[entities/MQTT|MQTT]] - MQTT 協定
- [[sources/factory-floor-digital-twin|factory-floor-digital-twin]] - 專案實作
- [[synthesis/Python-語法技巧|Python 語法技巧]] - 類型提示、執行緒鎖
