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

## [2026-04-18] add | 新增工業通訊協定與標準

補充工業自動化領域的通訊協定與國際標準。

### 新增頁面

**`synthesis/工業通訊協定與標準.md`**

### 通訊協定

| 協定 | 定位 | 特點 |
|------|------|------|
| MQTT | 雲端/邊緣 | 輕量、Pub/Sub |
| OPC UA | 工廠內部 | 安全、語義豐富 |
| Modbus | 現場設備 | 簡單、舊設備整合 |

### 國際標準

| 標準 | 名稱 | 內容 |
|------|------|------|
| ISA-95 / IEC 62264 | 自動化金字塔 | 五層架構定義 |
| ISA-88 / IEC 61512 | 批次控制 | 程序控制標準 |
| IEC 62443 | 工業資安 | IACS 安全等級 |
| ISO 23247 | 數位孿生製造 | DT 框架標準 |

### 2025 主流架構

OPC UA + MQTT 混合架構：
- OPC UA：工廠內部 M2M
- MQTT：邊緣到雲端

## [2026-04-19] update | 更新專案進度：Omniverse Extension

專案新增 Omniverse Extension，實現 MQTT 資料接收。

### 新增檔案

```
omniverse_extension/
├── config/extension.toml           # Extension 設定
└── omniverse_factory_twin/
    ├── __init__.py                 # 模組初始化
    └── extension.py                # Extension 主程式
```

### 新增功能

**FactoryTwinExtension**：
- 繼承 `omni.ext.IExt`
- `on_startup()` / `on_shutdown()` 生命週期
- MQTT 連線與訂閱機制
- `on_connect` / `on_message` 回呼處理

**Bridge 錯誤處理**：
- JSON 解析錯誤處理
- 發布異常處理

### 建立/更新頁面

**新增綜合分析**：
- `synthesis/Omniverse-Extension-開發.md`
  - Extension 檔案結構
  - extension.toml 設定說明
  - 生命週期方法
  - paho-mqtt 回呼簽名
  - 完整程式碼範例

**更新來源**：
- `sources/factory-floor-digital-twin.md`
  - 新增檔案結構
  - 新增 Omniverse Extension 程式碼說明
  - 更新架構圖
  - 新增 Omniverse API

### 架構更新

```
ROS2 Publisher → MQTT Bridge → Mosquitto → Omniverse Extension
                                              ↓
                                         3D 視覺化（待實作）
```

## [2026-04-19] update | 新增 paho-mqtt 回呼函式說明

在 `entities/MQTT.md` 中新增：

- `on_connect` 回呼函式簽名與用法
- `reason_code` 常見值對照表（MQTT 5.0）
- `on_message` 回呼函式簽名與用法

常用 reason_code：
- 0：Success（連線成功）
- 134：Bad user name or password
- 135：Not authorized
- 136：Server unavailable

## [2026-04-19] update | 新增 Omniverse Extension 常見問題

在 `synthesis/常見問題與解法.md` 中新增 Omniverse Extension 相關章節：

### 新增問題（4 個）

| 問題 | 原因 | 解法 |
|------|------|------|
| Extension Manager 找不到 Extension | `folders.'++'` 路徑未設定 | 加入 `${app}/../../../../source/extensions` |
| No module named 'paho' | Omniverse 使用獨立 Python 環境 | 用 Omniverse 的 python.exe 安裝 |
| MQTT WinError 10061 | Docker Mosquitto 未啟動 | `docker compose up -d` |
| Extension 與 Git 管理衝突 | 檔案分散兩個專案 | Windows Junction 連結 |

### 關鍵技巧

**Windows Junction**：
```powershell
mklink /J "目標路徑" "來源路徑"
```
- 類似 Linux symlink
- Extension 原始碼保留在專案中
- Git 追蹤不受影響

## [2026-04-22] update | 專案重構：Omniverse Extension 架構升級

專案進行重大重構，建立模組化的 Extension 架構。

### 新增檔案

```
omniverse_factory_twin/
├── mqtt_client.py      # MQTT 客戶端封裝
├── base_extension.py   # 抽象基底類別
└── extension.py        # 重構為繼承 BaseMqttExtension
```

### 架構變更

```
MqttClient（通訊封裝）
    ↑ 使用
BaseMqttExtension（抽象基底）
    ↑ 繼承
FactoryTwinExtension（實作）
    → USD Scene 更新
```

### 新增功能

| 功能 | 說明 |
|------|------|
| 執行緒安全更新 | `Lock + pendingUpdates` 模式 |
| USD 場景顏色變化 | 根據 status 更新機台顏色 |
| Hook 方法模式 | `onExtensionStartup()` / `onExtensionShutdown()` |
| 抽象方法 | `onMqttMessage()` 強制子類別實作 |

### 更新頁面

**`sources/factory-floor-digital-twin.md`**：
- 更新檔案結構
- 新增 mqtt_client.py、base_extension.py 完整程式碼
- 更新 extension.py 為重構後版本
- 新增 USD/pxr API 說明
- 更新已完成清單

**`synthesis/Omniverse-Extension-開發.md`**：
- 新增三層架構說明
- 新增執行緒安全模式
- 新增 USD API 快速參考
- 新增設計模式摘要表

**`synthesis/Python-語法技巧.md`**：
- 新增類型提示（`list[str]`、`Optional[Callable]`）
- 新增執行緒安全（`threading.Lock`）
- 新增抽象方法模式（`NotImplementedError`）
- 新增安全屬性檢查（`hasattr`）
- 新增 `self.__class__.__name__`

### 新增 Python 語法

| 語法 | 範例 |
|------|------|
| `list[str]` | Python 3.9+ 泛型類型提示 |
| `Optional[Callable]` | 可選回呼類型 |
| `threading.Lock()` | 執行緒鎖 |
| `with self.lock_:` | 執行緒安全區塊 |
| `dict(d)` | 字典淺複製 |
| `hasattr(obj, 'attr')` | 安全屬性檢查 |
| `raise NotImplementedError` | 抽象方法 |
| `self.__class__.__name__` | 取得類別名稱 |

### 新增 USD API

| API | 用途 |
|-----|------|
| `omni.usd.get_context().get_stage()` | 取得 Stage |
| `stage.GetPrimAtPath()` | 取得 Prim |
| `prim.IsValid()` | 檢查存在 |
| `UsdGeom.Gprim(prim)` | 轉為幾何物件 |
| `GetDisplayColorAttr().Set()` | 設定顏色 |
| `Gf.Vec3f(r, g, b)` | 顏色向量 |

## [2026-04-22] add | 新增 Omniverse Extension 執行緒問題

在 `synthesis/常見問題與解法.md` 中新增 4 個 Omniverse 開發問題：

| 問題 | 原因 | 解法 |
|------|------|------|
| ChangeProperty 執行失敗 | commands 不能在背景執行緒呼叫 | 改用 `UsdGeom.Gprim().GetDisplayColorAttr().Set()` |
| 顏色沒有即時更新 | `push()` 在某些 Kit 版本無效 | 改用 `create_subscription_to_pop()` |
| `lock_` 找不到 | 方法簽名不匹配導致覆寫失敗 | 統一 `onExtensionStartup(self, ext_id)` |
| 重啟後重複 subscribe | `on_shutdown` 未釋放連線 | 加入 `mqttClient_.disconnect()` |

### 關鍵教訓

1. **執行緒安全**：USD 操作必須在主執行緒，MQTT 在背景執行緒
2. **方法覆寫**：子類別方法簽名必須完全匹配父類別
3. **資源清理**：`on_shutdown()` 必須釋放所有資源

## [2026-05-03] update | 更新專案進度（04-22 → 04-29）

根據 GitHub 倉庫最新 commit 更新知識庫，涵蓋 04-22 到 04-29 的變更。

### 新增功能

**1. config 組態模組**
- `config/config_loader.py` - FactoryConfig、MachineConfig 類別
- `config/machines.toml` - 4 台機台設定（新增 machine_04）
- `config/thresholds.toml` - 閾值設定（溫度、振動）
- `config/topic_resolver.py` - Topic 命名規則統一管理

**2. 資料產生器重構**
- `ros2_publisher/topic_data_generator.py` - MachineState、ScriptPhase
- 支援腳本式狀態變化（模擬異常流程）
- machine_04 使用預定義腳本

**3. Topic 分割**
- 舊：`/factory/machine_01/status`（所有資料一個 Topic）
- 新：每參數獨立 Topic
  - `/factory/machine_01/temperature`
  - `/factory/machine_01/vibration`
  - `/factory/machine_01/operation_mode`

**4. 日誌記錄系統**
- `omniverse_extension/factory_log.py` - FactoryLog、MachineLog
- 固定大小循環緩衝區（deque maxlen=50）
- 支援查詢最新參數值

**5. 嚴重程度計算**
- 根據溫度/振動值計算 NORMAL/WARNING/ERROR
- 溫度：70°C warning, 85°C error
- 振動：5.0 warning, 10.0 error

**6. 視覺反饋強化**
- 透明度控制（RUNNING=1.0, IDLE=0.6, SHUTDOWN=0.4, OFFLINE=0.3）
- 顏色根據嚴重程度變化

**7. 外部資產**
- `external_assets/USD_Explorer_Sample_NVD@10011/`

### 更新的知識庫頁面

**來源**：
- `sources/factory-floor-digital-twin.md` - 全面更新

**綜合分析**：
- `synthesis/Omniverse-Extension-開發.md` - 新增日誌系統、嚴重程度
- `synthesis/Python-語法技巧.md` - 新增 dataclass、deque、TOML

### 新增 Commit 列表

| 日期 | SHA | 說明 |
|------|-----|------|
| 04-25 | cab9b1c | 新增 config 模組 |
| 04-25 | 6e916bb | Publisher 使用 config |
| 04-25 | 635f17e | Bridge 使用 config |
| 04-26 | fefcc22 | Topic 分割 |
| 04-26 | b44223a | Extension 使用 config |
| 04-26 | cb1bcee | 新增外部資產 |
| 04-28 | 94dab78 | 修正資料模擬邏輯 |
| 04-28 | 9d6213c | 新增 machine_04 |
| 04-28 | fe2aa81 | 修正 Topic 覆寫 bug |
| 04-29 | 1dee3d8 | 新增日誌系統 |
| 04-29 | beb9187 | Extension 整合日誌 |
| 04-29 | fbfc8e5 | 修正欄位錯誤 |

### 新增 Python 語法

| 語法 | 說明 |
|------|------|
| `@dataclass` | 資料類別裝飾器 |
| `deque(maxlen=N)` | 固定大小佇列 |
| `tomllib` / `tomli` | TOML 解析 |
| `tuple[str, str] \| None` | 新式 Union 語法 |
