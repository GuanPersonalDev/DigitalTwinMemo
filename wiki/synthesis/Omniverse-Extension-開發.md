---
title: Omniverse Extension 開發
type: synthesis
created: 2026-04-19
updated: 2026-04-19
sources: [factory-floor-digital-twin]
tags: [Omniverse, Extension, Python, 開發]
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
    └── extension.py        # 主程式
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

### __init__.py

```python
from .extension import FactoryTwinExtension
```

### extension.py

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
            print("[Factory Twin] Connect success")
            for topic in MACHINE_TOPICS:
                client.subscribe(topic)
                print(f"[Factory Twin] Subscribe: {topic}")
        else:
            print(f"[Factory Twin] Connect fail: {reason_code}")

    def onMqttMessage(self, client, userdata, msg):
        try:
            data = json.loads(msg.payload.decode())
            print(f"[Factory Twin] {msg.topic} -> {data}")
            # TODO: 更新 3D 場景
        except Exception as e:
            print(f"[Factory Twin] Error: {e}")
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

### 場景操作（未來擴充）

```python
from pxr import Usd, UsdGeom

# 取得目前 stage
stage = omni.usd.get_context().get_stage()

# 取得 prim（場景物件）
prim = stage.GetPrimAtPath("/World/Machine_01")

# 修改屬性
prim.GetAttribute("xformOp:translate").Set((0, 0, 0))
```

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

## 下一步

- [ ] 連接 3D 場景物件
- [ ] 根據溫度變更材質顏色
- [ ] 根據振動播放動畫
- [ ] 加入 UI 面板顯示即時數據
- [ ] 實作告警視覺效果

## 相關頁面

- [[entities/NVIDIA Omniverse|NVIDIA Omniverse]] - 平台介紹
- [[entities/MQTT|MQTT]] - MQTT 協定
- [[sources/factory-floor-digital-twin|factory-floor-digital-twin]] - 專案實作
