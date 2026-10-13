# 🌱 GardenControl-io

[![Home Assistant Add-on](https://img.shields.io/badge/Home%20Assistant-Add--on-blue.svg)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Architecture](https://img.shields.io/badge/arch-amd64%20%7C%20aarch64%20%7C%20armv7-green.svg)](https://github.com/Paradigm-Paradox/GardenControl-io)

**GardenControl-io** is an intelligent, edge-native garden orchestration platform built as a Home Assistant Add-on. It combines biological crop planning, companion planting intelligence, microclimate monitoring, and smart adaptive irrigation into a unified system for homesteads and gardens.

---

## 🌟 Key Features

- **🌾 Adaptive Evapotranspiration ($ET_0$) Irrigation**: Calculates daily crop water loss based on local temperature, solar radiation, relative humidity, and stage-specific crop coefficients ($K_c$).
- **💧 Cycle & Soak Runoff Prevention**: Breaks watering into pulsed intervals separated by soak periods to maximize percolation and prevent surface pooling on dense soils.
- **🗺️ Interactive Garden Bed Mapping**: Visual grid layout for raised beds, in-ground rows, and container pots with live soil moisture heatmaps.
- **🍅 Biological Crop Lifecycle Tracker**: Track days-to-maturity, growth stages, and companion planting compatibility (beneficial pairings and antagonist warnings).
- **🏡 Home Assistant Native**: Full Ingress dashboard in the HA sidebar, automatic MQTT entity discovery (`valves`, `sensors`, `switches`), and Supervisor API coordination.
- **⚡ Homestead Ecosystem Aware**: Native interlocks with **SoilSense-io** (soil sensors), **WaterTankControl-io** (cistern water levels), and **SolarControl-io** (solar pump synchronization).

---

## 📐 System Architecture

```text
               +-------------------------------------------+
               |         Home Assistant Supervisor         |
               |  (Ingress Proxy, Core API, MQTT Broker)   |
               +---------------------+---------------------+
                                     |
                       +-------------v-------------+
                       |     GardenControl-io      |
                       |       (Port 8099)         |
                       +-------------+-------------+
                                     |
         +---------------------------+---------------------------+
         |                           |                           |
+--------v---------+       +---------v---------+       +---------v---------+
|  Irrigation &    |       |  Biological Crop  |       |  HA & Ecosystem   |
|  ET Engine       |       |  & Bed Manager    |       |  Interlocks       |
|  - Cycle & Soak  |       |  - Companion Matrix|      |  - SoilSense-io   |
|  - Run Watchdog  |       |  - Stage Tracking |       |  - Solar Sync     |
+------------------+       +-------------------+       +-------------------+
```

For complete architectural diagrams and design specifications, see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/DESIGN.md](docs/DESIGN.md).

---

## 🚀 Installation

1. In Home Assistant, navigate to **Settings** > **Add-ons** > **Add-on Store**.
2. Click the three dots in the top right corner and select **Repositories**.
3. Add the repository URL:
   ```text
   https://github.com/Paradigm-Paradox/GardenControl-io
   ```
4. Find **GardenControl-io** in the add-on store, click **Install**, then **Start**.
5. Enable **Show in sidebar** to access the Ingress dashboard.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
