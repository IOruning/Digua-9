[English](README.md) | 中文

# 地瓜⑨号 · 水面搜救辅助系统

创新竞赛"机器人+ 安全应急和极限环境应用"参赛项目，场景为水面搜救辅助：无人机在指定水域巡航，把实时画面、飞控状态、AI 目标识别结果和智能手环风险告警统一呈现给救援指挥人员和比赛评委。

本仓库是项目的**主仓库**，通过 Git Submodules 聚合本项目的开源部分。

## 整体系统架构

完整比赛系统由四个子系统组成：

```text
┌──────────────┐   BLE 广播    ┌───────────────────┐   独立 MQTT/WSS   ┌───────────────┐
│   智能手环     │ ────────────▶ │    RDK 网关         │ ─────────────────▶│               │
│               │  water_contact│  BLE 扫描/去重       │  手环风险事件、区域   │               │
│               │  IMU/SOS 等   │  风险判断/事件生成    │  级定位、网关状态     │               │
└──────────────┘               │  局域网管理端         │                   │  web-client    │
                                └───────────────────┘                    │               │
┌──────────────┐   MAVLink     ┌───────────────────┐   同一条 WebRTC     │  浏览器只读展示   │
│     PX4       │ ────────────▶ │  rdk-x5-webrtc     │  会话(视频+遥测+   │               │
│    飞控        │               │                     │  视觉检测+投放状态) │               │
└──────────────┘               │  机载 C++ WebRTC 服务 │ ─────────────────▶│               │
┌──────────────┐   USB/文件     │  DOSOD 视觉识别       │                   └───────────────┘
│  摄像头/测试视频 │ ────────────▶ │                     │
└──────────────┘               └───────────────────┘
```

| 子系统 | 职责 |
| --- | --- |
| **web-client** | 浏览器只读前端，展示视频、遥测、视觉识别、救援事件 |
| **rdk-x5-webrtc** | 机载 C++ 服务：WebRTC 图传、MAVLink 遥测桥接、DOSOD 视觉识别 |

前端（web-client）通过独立的 MQTT/WSS 只读订阅 RDK 网关发布的业务事件，两个部分均可独立编译、独立演示（机载端的本地视频分支 + 前端的模拟遥测模式）。

前端与机载服务严格只读：不包含飞控指令、投放控制、SOS 发起或事件处置回写能力；这些动作由遥控器、飞控、QGroundControl 或 RDK 网关本地管理端负责。

## 仓库结构

```text
nodehub/
├── rdk-x5-webrtc/   # submodule：机载 C++ WebRTC/遥测/视觉服务
├── web-client/      # submodule：浏览器前端
└── README.md
```

## 获取代码

```bash
git clone --recurse-submodules <本仓库地址>
```

如果已经 clone 过忘了带 `--recurse-submodules`：

```bash
git submodule update --init --recursive
```

## 快速看效果（不需要任何硬件）

```bash
cd web-client
npm install
npm run dev
```

打开 `http://localhost:5173/`，在调试区开启模拟遥测（`VITE_ENABLE_TELEMETRY_SIMULATION=true`），即可看到完整界面和数据流转效果，无需无人机、RDK X5 板子或 MQTT broker。

如果有 RDK X5 硬件，`rdk-x5-webrtc` 支持用仓库自带的示例视频跑通"本地视频 → WebRTC 推流"链路，具体步骤见 [`rdk-x5-webrtc/README.md`](rdk-x5-webrtc/README.md)。

## 各子模块文档

- [`rdk-x5-webrtc/README.md`](rdk-x5-webrtc/README.md)：机载服务架构、编译部署、协议契约
- [`web-client/README.md`](web-client/README.md)：前端架构、开发调试、部署方式

## 许可证

主仓库及两个子模块均使用 [Apache License 2.0](LICENSE)。`rdk-x5-webrtc` 基于 [TzuHuanTai/RaspberryPi-WebRTC](https://github.com/TzuHuanTai/RaspberryPi-WebRTC) 二次开发，详见其 LICENSE 文件。
