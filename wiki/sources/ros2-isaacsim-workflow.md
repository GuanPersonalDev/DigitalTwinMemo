---
title: ROS2/Isaac Sim 工作流程教學
type: source
created: 2026-04-17
updated: 2026-04-17
original: https://samyooole.github.io/blog/jekyll/2025/01/11/isaacsim.html
tags: [教學, ROS2, Isaac Sim, 數位孿生, 模擬]
---

# ROS2/Isaac Sim 工作流程教學

**作者**：Samuel Ho
**日期**：2025-01-11
**類型**：實作教學

## 核心觀點

本教學展示如何建立 ROS2 與 Isaac Sim 的整合工作流程，從匯入 3D 機器人模型到撰寫控制程式碼，實現「先模擬、後部署」的開發策略。

## 系統需求

| 項目 | 要求 |
|------|------|
| 作業系統 | Ubuntu 22.04（必須） |
| ROS2 版本 | Humble |
| 啟動前置 | `source /opt/ros/humble/setup.bash` |

**重要**：避免使用 Windows/WSL，會有過多的相容性問題。

## 工作流程比較

```
實體開發（橘色）：組裝機器人 → 撰寫控制迴圈 → 實體測試
模擬開發（紅色）：虛擬關節設定 → ROS2 程式碼 → Isaac Sim 測試
                                    ↓
                             Sim2Real 轉移
```

**目標**：讓模擬行為與真實機器人一致，在部署前快速迭代。

## 七步驟實作流程

### 1. 安裝與設定

安裝 Isaac Sim，選擇正確的 Ubuntu 和 ROS2 版本。

### 2. 環境與資產設定

```bash
# 從 Omniverse 匯入基本地板和環境模板
# Clone 機器人描述檔（以 TurtleBot4 為例）

# 將 .urdf.xacro 轉換為 .urdf
ros2 run xacro xacro -o <filename>.urdf <filename>.urdf.xacro
```

### 3. 參數調整

調整物理屬性（如質量）確保機器人在模擬中正確平衡與移動。

### 4. 馬達關節設定（Action Graphs）

使用視覺化腳本工具設定馬達控制：

| 參數 | 值 |
|------|-----|
| target primitives | 機器人 base link |
| joint names | left_wheel_joint, right_wheel_joint |
| maxLinearSpeed | 0.22 |
| wheelDistance | 0.16 |
| wheelRadius | 0.025 |

測試指令：
```bash
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}, angular: {z: 0.0}}"
```

### 5. 感測器整合

相機設定流程：
1. 建立 camera 物件並 parent 到適當的 frame
2. 調整方向（X=90, Y=-90 朝前）
3. 設定 Action Graph：render products + ROS2 Camera Helper
4. 發布到指定 topic（如 `/rgb`）

### 6. ROS2 Package 建立

```bash
# 建立 package
ros2 pkg create --build-type ament_cmake --node-name hello_robot_node hello_robot_package

# 更新 package.xml 和 CMakeLists.txt（或 Python 的 setup.py）
```

### 7. 控制程式碼實作

作者實作了一個追蹤人類的有限狀態機：
- 主邏輯：C++ (hello_robot_node.cpp)
- 人類偵測：Python (human_det_node.py)
- 狀態轉換基於偵測結果

### 建置與執行

```bash
source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash
ros2 run [package_name] [node_name]
```

## 關鍵概念

| 概念 | 說明 |
|------|------|
| Digital Twins | 高擬真虛擬複製品，可安全測試不損耗實體 |
| Action Graphs | 視覺化腳本介面，將虛擬關節控制映射到 ROS topic |
| URDF | XML 格式的機器人描述檔（關節、連結、物理屬性） |
| Sim2Real | 在模擬中驗證演算法後再部署到實體機器人 |

## 與我的專案的關聯

這篇教學可以直接應用到 [[sources/factory-floor-digital-twin|factory-floor-digital-twin]] 專案：

1. **現有**：ROS2 Publisher 發布機台狀態
2. **下一步**：在 Isaac Sim 中建立工廠 3D 場景
3. **整合**：用 ROS2 Bridge 將模擬資料視覺化

```
factory_publisher.py (現有)
       ↓ ROS2 Topic
Isaac Sim ROS2 Bridge (新增)
       ↓
3D 工廠數位孿生視覺化
```

## 實作建議

1. **環境一致性**：務必使用 Ubuntu 22.04 + ROS2 Humble
2. **漸進測試**：先測試馬達、再測試感測器，最後整合
3. **命名規範**：joint 和 topic 使用描述性名稱便於除錯
4. **啟動順序**：先 source ROS 再開啟 Isaac Sim

## 相關頁面

- [[entities/NVIDIA Isaac|NVIDIA Isaac]] - Isaac 平台介紹
- [[entities/ROS2|ROS2]] - ROS2 指令參考
- [[synthesis/ROS2-Python-模式|ROS2 Python 模式]] - Python 程式碼模式
- [[concepts/數位孿生|數位孿生]] - 概念說明

## 來源

- [原文連結](https://samyooole.github.io/blog/jekyll/2025/01/11/isaacsim.html)
