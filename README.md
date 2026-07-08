English | [中文](README_cn.md)

# Digua-9 · Water-Rescue Assistance System

An entry for the "Robot+ Safety Emergency and Extreme Environment Applications" innovation competition. Scenario: a drone patrols a designated body of water, streaming live video, flight status, AI object-detection results, and smart-wristband risk alerts to rescue commanders and competition judges in one unified view.

This repository is the project's **main repository**, aggregating the open-source portion of the project via Git Submodules.

## Overall system architecture

The full competition system has four subsystems:

```text
┌──────────────┐  BLE advert.  ┌───────────────────┐  Separate MQTT/WSS ┌───────────────┐
│  Wristband    │ ────────────▶ │    RDK Gateway      │ ─────────────────▶│               │
│ (not open-    │  water_contact│  BLE scan/dedup      │  wristband risk    │               │
│  sourced)     │  IMU/SOS etc. │  risk判断/event gen   │  events, zone-level │               │
└──────────────┘               │  LAN operator console │  location, gateway  │  web-client    │
                                │ (not open-sourced)   │  status             │  (submodule ·   │
                                └───────────────────┘                       │   open-sourced) │
┌──────────────┐   MAVLink     ┌───────────────────┐   Single WebRTC       │  browser        │
│     PX4       │ ────────────▶ │  rdk-x5-webrtc     │  session (video +   │  read-only UI   │
│ flight ctrl   │               │  (submodule ·       │  telemetry + vision │               │
└──────────────┘               │   open-sourced)     │  detections +       │               │
┌──────────────┐  USB/file      │  onboard C++ WebRTC  │  release status)    │               │
│ Camera / test  │ ───────────▶ │  service + DOSOD      │ ─────────────────▶│               │
│ video          │              │  vision detection    │                    └───────────────┘
└──────────────┘               └───────────────────┘
```

| Subsystem | Responsibility | Status in this repo |
| --- | --- | --- |
| **web-client** | Read-only browser frontend: video, telemetry, vision detections, rescue events | ✅ Open-sourced (submodule) |
| **rdk-x5-webrtc** | Onboard C++ service: WebRTC video, MAVLink telemetry bridge, DOSOD vision detection | ✅ Open-sourced (submodule) |
| RDK Gateway | BLE scanning/dedup, risk assessment, rescue-event generation, LAN operator console | Not included in this repo |
| Wristband firmware | Collects water-contact, IMU, SOS, battery status and reports over BLE | Not included in this repo |

The frontend (web-client) subscribes read-only to business events published by the RDK Gateway over a separate MQTT/WSS link. So even though the gateway and wristband subsystems' code is not in this repository, the two open-sourced parts — frontend and onboard service — can each be built and demoed independently (the onboard service's local-video branch, and the frontend's simulated-telemetry mode).

The website and onboard service are strictly read-only: no flight-control commands, release control, SOS initiation, or event-disposition write-back. Those actions are handled by the remote control, the flight controller, QGroundControl, or the RDK Gateway's local operator console.

## Repository layout

```text
nodehub/
├── rdk-x5-webrtc/   # submodule: onboard C++ WebRTC/telemetry/vision service
├── web-client/      # submodule: browser frontend
└── README.md
```

## Getting the code

```bash
git clone --recurse-submodules <this repo's URL>
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## See it running in minutes (no hardware needed)

```bash
cd web-client
npm install
npm run dev
```

Open `http://localhost:5173/` and enable simulated telemetry in the debug panel (`VITE_ENABLE_TELEMETRY_SIMULATION=true`) to see the full UI and data flow — no drone, RDK X5 board, or MQTT broker required.

If you have RDK X5 hardware, `rdk-x5-webrtc` can run the "local video → WebRTC streaming" path using the bundled sample video; see [`rdk-x5-webrtc/README.md`](rdk-x5-webrtc/README.md) for the steps.

## Submodule documentation

- [`rdk-x5-webrtc/README.md`](rdk-x5-webrtc/README.md): onboard service architecture, build/deploy, protocol contracts
- [`web-client/README.md`](web-client/README.md): frontend architecture, development, deployment

## License

The main repository and both submodules are licensed under [Apache License 2.0](LICENSE). `rdk-x5-webrtc` is a derivative work of [TzuHuanTai/RaspberryPi-WebRTC](https://github.com/TzuHuanTai/RaspberryPi-WebRTC); see its LICENSE file for details.
