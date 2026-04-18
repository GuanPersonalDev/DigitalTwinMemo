---
title: Docker
type: entity
created: 2026-04-18
updated: 2026-04-18
sources: [factory-floor-digital-twin]
tags: [工具, 容器, 部署]
---

# Docker

容器化平台，用於打包和部署應用程式。

## 基本資訊

- **類型**：容器化平台
- **用途**：應用程式打包、部署、隔離執行
- **官網**：https://www.docker.com

## Docker Compose

用於定義和運行多容器應用的工具，使用 YAML 檔案設定。

### 基本結構

```yaml
services:
  服務名稱:
    image: 映像檔名稱:標籤
    container_name: 容器名稱
    ports:
      - "主機端口:容器端口"
    volumes:
      - 主機路徑:容器路徑
    restart: 重啟策略
```

### 專案實例（factory-floor-digital-twin）

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto:2.0
    container_name: factory_mosquitto
    ports:
      - "1883:1883"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
    restart: unless-stopped
```

### 設定說明

| 設定 | 說明 | 範例 |
|------|------|------|
| `image` | Docker 映像檔 | `eclipse-mosquitto:2.0` |
| `container_name` | 容器名稱，方便識別 | `factory_mosquitto` |
| `ports` | 端口映射 `主機:容器` | `"1883:1883"` |
| `volumes` | 目錄掛載 `主機:容器` | `./config:/mosquitto/config` |
| `restart` | 重啟策略 | `unless-stopped` |

### volumes 掛載

將主機目錄映射到容器內，實現：
- **設定檔持久化**：容器重建後設定不遺失
- **資料持久化**：資料儲存在主機上
- **日誌存取**：方便從主機查看日誌

```
主機目錄                    容器目錄              用途
─────────────────────────────────────────────────────
./mosquitto/config    →    /mosquitto/config    設定檔
./mosquitto/data      →    /mosquitto/data      持久化資料
./mosquitto/log       →    /mosquitto/log       日誌檔案
```

### restart 重啟策略

| 值 | 說明 |
|----|------|
| `no` | 不自動重啟（預設） |
| `always` | 總是重啟（包括手動停止後重開 Docker） |
| `on-failure` | 只在錯誤退出時重啟 |
| `unless-stopped` | 除非手動停止，否則自動重啟（推薦） |

## 常用指令

### Docker Compose

```bash
# 啟動服務（背景執行）
docker-compose up -d

# 查看運行中的服務
docker-compose ps

# 查看日誌
docker-compose logs 服務名稱
docker-compose logs -f 服務名稱  # 持續追蹤

# 停止服務
docker-compose down

# 重啟服務
docker-compose restart

# 重建並啟動（設定變更後）
docker-compose up -d --build
```

### Docker 基本指令

```bash
# 列出運行中的容器
docker ps

# 列出所有容器（含已停止）
docker ps -a

# 進入容器內部
docker exec -it 容器名稱 /bin/sh

# 查看容器日誌
docker logs 容器名稱

# 停止容器
docker stop 容器名稱

# 移除容器
docker rm 容器名稱
```

## 與數位孿生的關係

Docker 在數位孿生專案中的角色：

```
┌─────────────────────────────────────┐
│  Docker Compose                     │
│  ┌───────────┐  ┌───────────┐      │
│  │ Mosquitto │  │ 其他服務  │      │
│  │  (MQTT)   │  │ (未來)    │      │
│  └─────┬─────┘  └───────────┘      │
└────────┼────────────────────────────┘
         │ port 1883
         ↓
   ros2_to_mqtt.py
         ↑
   ROS2 Topics
```

## 目錄結構建議

```
project/
├── docker-compose.yml
├── mosquitto/
│   ├── config/
│   │   └── mosquitto.conf
│   ├── data/          # 自動產生
│   └── log/           # 自動產生
└── ...
```

## 相關實體

- [[MQTT]] - Mosquitto MQTT Broker
- [[ROS2]] - 機器人作業系統

## 來源引用

- [[sources/factory-floor-digital-twin|factory-floor-digital-twin]] - Docker 部署實例
