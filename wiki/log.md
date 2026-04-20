---
title: 操作日誌
type: log
created: 2026-04-16
---

# 操作日誌

按時間順序記錄知識庫的所有操作。

---

## [2026-04-16] init | 知識庫初始化

- 建立目錄結構
- 建立 CLAUDE.md schema 檔案
- 建立 index.md 索引
- 建立 log.md 日誌
- 建立 overview.md 總覽

知識庫已準備就緒，等待第一個來源。

## [2026-04-16] config | 設定語言偏好

- 將所有內容更新為繁體中文
- 在 CLAUDE.md 加入語言偏好規範

## [2026-04-16] config | 定義研究主題

- 研究主題：數位孿生 × NVIDIA Omniverse
- 學習階段：初學者
- 資料範圍：技術實作 + 理論概念 + 案例研究

建立初始頁面：
- 更新 `wiki/overview.md` - 定義研究方向與核心問題
- 建立 `wiki/concepts/數位孿生.md` - 核心概念頁面
- 建立 `wiki/concepts/USD.md` - 技術概念頁面
- 建立 `wiki/entities/NVIDIA Omniverse.md` - 平台實體頁面
- 更新 `wiki/index.md` - 加入新頁面

等待第一個來源攝入。

## [2026-04-16] add | 新增 ROS2 指令參考

- 建立 `wiki/entities/ROS2.md` - ROS2 指令參考頁面
- 內容：topic、node、interface、service 相關指令
- 來源：個人練習筆記
- 更新 `wiki/index.md` - 加入 ROS2 頁面

此頁面設計為累積式參考，後續 ROS2 相關指令將持續加入。

## [2026-04-16] add | 新增 rclpy 函式庫與 Python 模式

來源：MachinePublisher 練習程式碼

建立頁面：
- `wiki/entities/rclpy.md` - rclpy 函式庫參考
  - 模組：rclpy, rclpy.node.Node
  - 訊息類型：std_msgs.msg
  - 函式與方法說明表格
- `wiki/synthesis/ROS2-Python-模式.md` - 程式碼模式參考
  - 基本節點結構
  - Publisher / Subscriber 模式
  - 多 Publisher 模式
  - JSON 資料格式

兩個頁面皆設計為累積式參考，後續相關程式碼將持續更新。

## [2026-04-16] config | 新增快捷指令

在 CLAUDE.md 中新增「快捷指令」區段：

新增內容指令：
- `+指令 <頁面>` - 新增指令到指定頁面
- `+函式 <頁面>` - 新增函式說明
- `+模式 <頁面>` - 新增程式碼模式
- `+概念 <名稱>` - 建立概念頁面
- `+實體 <名稱>` - 建立實體頁面
- `+來源 <路徑>` - 攝入來源

查詢維護指令：
- `?<問題>` - 查詢知識庫
- `!lint` - 健康度檢查
- `!index` - 顯示索引
- `!log` - 顯示日誌

## [2026-04-16] ingest | 批次攝入 5 篇數位孿生相關文章

### 來源清單

1. **Digital twin: Data exploration, architecture, implementation and future**
   - 類型：學術論文（PMC）
   - 重點：六大領域資料處理流程

2. **Digital twin driven smart factories (Edge AI + FL)**
   - 類型：學術論文（Nature）
   - 重點：邊緣運算 + 聯邦學習框架

3. **NVIDIA Omniverse: $50T Physical AI Operating System**
   - 類型：技術文章
   - 重點：Omniverse 企業應用與成效

4. **NVIDIA Unveils 'Mega' Omniverse Blueprint**
   - 類型：官方公告（CES 2025）
   - 重點：工業機器人車隊數位孿生

5. **Paving the Future of Factories with NVIDIA Omniverse**
   - 類型：案例研究
   - 重點：BMW 工廠數位孿生實踐

### 建立的頁面

**來源摘要（5 篇）**：
- `sources/digital-twin-data-exploration.md`
- `sources/digital-twin-edge-ai-federated-learning.md`
- `sources/nvidia-omniverse-50t-physical-ai.md`
- `sources/nvidia-mega-omniverse-blueprint.md`
- `sources/bmw-omniverse-enterprise.md`

**新增概念（2 個）**：
- `concepts/Edge AI.md`
- `concepts/Federated Learning.md`

**新增實體（1 個）**：
- `entities/NVIDIA Isaac.md`

**更新頁面**：
- `concepts/數位孿生.md` - 加入正式定義、架構、應用場景
- `entities/NVIDIA Omniverse.md` - 加入企業案例、技術需求、生態系統

### 關鍵發現

- 數位孿生的標準定義與歷史（NASA 阿波羅任務）
- Edge AI + FL 可降低延遲 35%、雲端使用 28%
- Omniverse 已有 252+ 企業部署，效率提升 30-70%
- BMW 31 間工廠使用 Omniverse，規劃效率提升 30%
- Mega Blueprint 為機器人車隊提供參考架構

## [2026-04-17] analyze | 分析 factory-floor-digital-twin 專案

來源：https://github.com/GuanPersonalDev/factory-floor-digital-twin

### 專案分析

ROS2 工廠機台模擬專案，使用 Python 和 rclpy 實作：
- 3 台機台的感測器資料模擬
- 定時發布到 ROS2 topic
- JSON 格式的狀態訊息

### 建立/更新頁面

**新增來源**：
- `sources/factory-floor-digital-twin.md` - 專案文件與架構分析

**新增綜合分析**：
- `synthesis/Python-語法技巧.md` - Python 語法模式
  - 字典推導式、f-string、round()、json 模組

**更新累積式頁面**：
- `entities/rclpy.md` - 加入專案實例程式碼
- `synthesis/ROS2-Python-模式.md` - 加入完整 MachinePublisher 範例

### 提取的 API 與語法

**ROS2/rclpy API**：
- `rclpy.init()`, `rclpy.spin()`, `rclpy.shutdown()`
- `Node.__init__()`, `create_publisher()`, `create_timer()`
- `get_logger().info()`, `publisher.publish()`

**Python 標準庫**：
- `json.dumps()` - 字典轉 JSON
- `random.uniform()`, `random.choice()` - 隨機數生成
- `round()` - 數值四捨五入

**Python 語法技巧**：
- 字典推導式建立多個 Publisher
- f-string 動態生成 topic 名稱
- `.items()` 遍歷字典

## [2026-04-17] ingest | 攝入 ROS2/Isaac Sim 工作流程教學

來源：https://samyooole.github.io/blog/jekyll/2025/01/11/isaacsim.html

### 選擇原因

根據目前學習狀態推薦：
- 已有 ROS2 Python 基礎（factory-floor-digital-twin 專案）
- 已了解數位孿生概念
- 下一步需要學習視覺化整合

此教學正好銜接 ROS2 與 Isaac Sim，是實現數位孿生視覺化的關鍵步驟。

### 建立/更新頁面

**新增來源**：
- `sources/ros2-isaacsim-workflow.md` - 完整七步驟工作流程

**更新實體**：
- `entities/NVIDIA Isaac.md` - 加入系統需求、ROS2 整合方式

### 關鍵內容

- 系統需求：Ubuntu 22.04 + ROS2 Humble
- 七步驟流程：安裝 → 環境 → 參數 → 馬達 → 感測器 → Package → 控制碼
- Action Graphs：視覺化腳本介面
- Sim2Real：先模擬後部署的策略

## [2026-04-18] update | 更新 factory-floor-digital-twin 專案進度

專案新增 ROS2 → MQTT 橋接功能。

### 新增檔案

```
bridge/
├── ros2_to_mqtt.py    # ROS2 訂閱者 → MQTT 發布
└── config.py          # 組態設定（broker、topic 對應）

mosquitto/
└── config/mosquitto.conf

docker-compose.yml     # Mosquitto MQTT Broker
```

### 架構更新

```
machine_publisher.py (ROS2 Publisher)
       ↓ ROS2 Topic
ros2_to_mqtt.py (Bridge)
       ↓ MQTT
Web Dashboard / Omniverse / 其他系統
```

### 建立/更新頁面

**新增實體**：
- `entities/MQTT.md` - MQTT 協定與 paho-mqtt 使用

**更新頁面**：
- `sources/factory-floor-digital-twin.md` - 新增 Bridge 架構與程式碼
- `entities/rclpy.md` - 新增 Subscription 使用與 lambda 技巧
- `synthesis/ROS2-Python-模式.md` - 新增橋接模式完整範例
- `synthesis/Python-語法技巧.md` - 新增 lambda 閉包陷阱與組態模組模式

### 新增 API/語法

**paho-mqtt**：
- `mqtt.Client()`, `connect()`, `loop_start()`, `publish()`

**Python 技巧**：
- Lambda 預設參數捕捉迴圈變數：`lambda msg, t=topic: fn(msg, t)`
- 組態模組模式：`from config import SETTINGS`

## [2026-04-18] update | 擴充 create_subscription 說明

在 `entities/rclpy.md` 中擴充 `create_subscription` 的詳細說明：
- 語法與參數說明
- 與 Publisher 的對比
- 執行流程圖
- 基本用法與動態建立範例
- 回呼函式簽名

## [2026-04-18] add | 新增 Docker 實體頁面與 Mosquitto 設定說明

### 新增頁面

- `entities/Docker.md` - Docker 與 Docker Compose 使用說明
  - docker-compose.yml 結構與設定
  - volumes 掛載說明
  - restart 重啟策略
  - 常用指令參考

### 更新頁面

- `entities/MQTT.md` - 擴充 mosquitto.conf 設定說明
  - 各設定項詳細說明表格
  - 正式環境設定參考（TLS、帳密驗證）

## [2026-04-18] add | 新增常見問題與解法

整理開發過程中遇到的問題與解決方案。

### 新增頁面

- `synthesis/常見問題與解法.md`

### 收錄問題類別

**Docker 相關（3 個）**：
- 找不到設定檔 → cd 到專案目錄
- Port already allocated → 停止舊容器
- 容器 Restarting → 檢查設定檔拼寫

**WSL2 網路（2 個）**：
- ConnectionRefusedError → 改用 Windows IP
- host.docker.internal 無法解析 → 從 resolv.conf 取得 IP

**Python/paho-mqtt（3 個）**：
- ModuleNotFoundError → pip install
- DeprecationWarning → 使用 CallbackAPIVersion.VERSION2
- import 錯誤 → 正確的 import 語法

**拼寫錯誤（2 個）**：
- statue vs status
- persistance vs persistence

## [2026-04-18] update | 更新 factory-floor-digital-twin 新進度

專案新增 MQTT 重連機制與優雅關閉功能。

### 新增功能

**1. MQTT 連線重試（connectMqtt）**：
- 最多重試 5 次
- 每次失敗等待 2 秒
- 超過次數拋出 RuntimeError

**2. 優雅關閉（destroy_node）**：
- 停止 MQTT 背景迴圈
- 斷開 MQTT 連線
- 呼叫父類別清理

**3. KeyboardInterrupt 處理**：
- try/except/finally 確保資源釋放
- Ctrl+C 不報錯

### 更新頁面

- `sources/factory-floor-digital-twin.md` - 新增程式碼與 API
- `synthesis/ROS2-Python-模式.md` - 新增連線重試與優雅關閉模式
- `entities/rclpy.md` - 新增 destroy_node、rclpy.ok()

### 新增 API

| API | 說明 |
|-----|------|
| `Node.destroy_node()` | 節點清理（可覆寫） |
| `rclpy.ok()` | 檢查 ROS2 是否運行中 |
| `mqtt.loop_stop()` | 停止 MQTT 背景迴圈 |
| `mqtt.disconnect()` | 斷開 MQTT 連線 |

## [2026-04-18] add | 新增應用領域知識

補充數位孿生的應用領域背景知識。

### 新增概念頁面

**`concepts/工業4.0.md`**：
- 工業革命演進（1.0 → 4.0）
- 九大技術支柱
- 智慧工廠架構
- 預測性維護應用
- 投資趨勢數據

**`concepts/智慧製造.md`**：
- 與傳統製造的差異
- 四大核心要素（連接、數據、分析、自動化）
- 關鍵績效指標（OEE、MTBF、MTTR）
- 成功案例（台積電、BMW）
- 導入階段對照

### 新增綜合分析

**`synthesis/工廠感測器與監測指標.md`**：
- 溫度感測器：異常判斷、業界標準
- 振動感測器：ISO 10816 標準
- 狀態指示：建議擴充狀態
- 其他感測器：電流、壓力、轉速
- 告警閾值設計範例
- 資料格式建議

### 關鍵知識

| 主題 | 內容 |
|------|------|
| 預測性維護 | 比反應式/預防式更優化 |
| OEE | 可用率 × 效能 × 良率 |
| 振動標準 | ISO 10816，< 4.5 mm/s 為可接受 |
| 溫度警告 | 70-85°C 為警告區間 |
