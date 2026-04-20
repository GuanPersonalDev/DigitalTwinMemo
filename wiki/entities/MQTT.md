---
title: MQTT
type: entity
created: 2026-04-18
updated: 2026-04-19
sources: [factory-floor-digital-twin]
tags: [協定, IoT, 訊息佇列]
---

# MQTT

Message Queuing Telemetry Transport

## 基本資訊

- **類型**：輕量級訊息協定
- **用途**：IoT 裝置間通訊
- **預設 Port**：1883（未加密）、8883（TLS）
- **模式**：發布/訂閱（Pub/Sub）

## 核心概念

### 架構

```
Publisher ──→ Broker ──→ Subscriber
              (中介)
```

### 組件

| 組件 | 說明 |
|------|------|
| Broker | 中央伺服器，負責訊息路由 |
| Publisher | 發布訊息到指定 topic |
| Subscriber | 訂閱 topic 接收訊息 |
| Topic | 訊息分類路徑（如 `factory/machine_01/status`） |

### Topic 命名

使用 `/` 分隔層級，支援萬用字元：
- `+` 單層萬用：`factory/+/status` 匹配任一機台
- `#` 多層萬用：`factory/#` 匹配 factory 下所有

## Python 使用（paho-mqtt）

### 安裝

```bash
pip install paho-mqtt
```

### Publisher 範例

```python
import paho.mqtt.client as mqtt

client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
client.connect("localhost", 1883)
client.loop_start()

# 發布訊息
client.publish("factory/machine_01/status", '{"temperature": 75.5}')
```

### Subscriber 範例

```python
def on_message(client, userdata, msg):
    print(f"{msg.topic}: {msg.payload.decode()}")

client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
client.on_message = on_message
client.connect("localhost", 1883)
client.subscribe("factory/#")
client.loop_forever()
```

### 回呼函式（Callbacks）

#### on_connect

連線結果回呼，用於確認連線狀態：

```python
def on_connect(client, userdata, flags, reason_code, properties):
    """
    client:      mqtt.Client 實例
    userdata:    使用者自訂資料
    flags:       連線旗標（dict）
    reason_code: 連線結果代碼（0=成功）
    properties:  MQTT 5.0 屬性
    """
    if reason_code == 0:
        print("連線成功")
        client.subscribe("factory/#")
    else:
        print(f"連線失敗: {reason_code}")

client.on_connect = on_connect
```

#### reason_code 常見值（MQTT 5.0）

| 代碼 | 名稱 | 說明 |
|------|------|------|
| **0** | Success | 連線成功 |
| 128 | Unspecified error | 未指定錯誤 |
| 129 | Malformed packet | 封包格式錯誤 |
| 130 | Protocol error | 協定錯誤 |
| 132 | Unsupported protocol version | 不支援的協定版本 |
| 133 | Client identifier not valid | 無效的 Client ID |
| 134 | Bad user name or password | 帳號或密碼錯誤 |
| 135 | Not authorized | 未授權 |
| 136 | Server unavailable | 伺服器不可用 |
| 137 | Server busy | 伺服器忙碌 |
| 138 | Banned | 被封禁 |
| 159 | Connection rate exceeded | 連線頻率過高 |

#### on_message

收到訊息回呼：

```python
def on_message(client, userdata, msg):
    """
    client:   mqtt.Client 實例
    userdata: 使用者自訂資料
    msg:      訊息物件
        - msg.topic:   topic 名稱（str）
        - msg.payload: 訊息內容（bytes）
    """
    data = json.loads(msg.payload.decode())
    print(f"{msg.topic}: {data}")

client.on_message = on_message
```

## Docker 部署（Mosquitto）

### docker-compose.yml

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto:2.0
    container_name: factory_mosquitto
    ports:
      - "1883:1883"
    volumes:
      - ./mosquitto/config:/mosquitto/config
```

### mosquitto.conf

```conf
listener 1883
allow_anonymous true
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
```

#### 設定說明

| 設定 | 值 | 說明 |
|------|-----|------|
| `listener` | `1883` | 監聽端口，MQTT 標準端口 |
| `allow_anonymous` | `true` | 允許匿名連線（無需帳密） |
| `persistence` | `true` | 啟用持久化，重啟後保留訊息 |
| `persistence_location` | `/mosquitto/data/` | 持久化資料儲存路徑 |
| `log_dest` | `file ...` | 日誌輸出到檔案 |

#### 正式環境設定（參考）

```conf
# 基本設定
listener 1883

# 安全設定（停用匿名，啟用帳密）
allow_anonymous false
password_file /mosquitto/config/passwd

# TLS 加密
listener 8883
certfile /mosquitto/certs/server.crt
keyfile /mosquitto/certs/server.key

# 持久化與日誌
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
log_type all
```

### 啟動與測試

```bash
# 啟動
docker-compose up -d

# 訂閱測試
mosquitto_sub -h localhost -t "factory/#"

# 發布測試
mosquitto_pub -h localhost -t "factory/test" -m "hello"
```

## 與 ROS2 的整合

在數位孿生架構中，MQTT 作為 ROS2 與外部系統的橋樑：

```
ROS2 Topic ←→ Bridge ←→ MQTT ←→ Web/Cloud/其他系統
```

參見：[[sources/factory-floor-digital-twin|factory-floor-digital-twin]] 專案的 `ros2_to_mqtt.py`

## 與數位孿生的關係

MQTT 在數位孿生中的角色：
1. **邊緣到雲端**：將感測器資料傳輸到雲端平台
2. **跨系統整合**：連接不同協定的系統（ROS2、Web、PLC）
3. **低頻寬環境**：適合工廠網路受限場景

## 相關實體

- [[ROS2]] - 機器人作業系統
- [[Docker]] - 容器化部署
- [[concepts/Edge AI|Edge AI]] - 邊緣運算

## 來源引用

- [[sources/factory-floor-digital-twin|factory-floor-digital-twin]] - ROS2 to MQTT Bridge 實作
