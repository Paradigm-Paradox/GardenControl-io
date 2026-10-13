# GardenControl-io: Design Specification

This document provides a concise overview of the **GardenControl-io** system architecture and functional design. For the full deep-dive specifications, see [docs/DESIGN.md](file:///mnt/data/GardenControl-io/docs/DESIGN.md) and [docs/ARCHITECTURE.md](file:///mnt/data/GardenControl-io/docs/ARCHITECTURE.md).

## Core Capabilities
1. **Adaptive Irrigation & Evapotranspiration ($ET_0$)**: Dynamic water budgeting factoring in soil type, temperature, solar radiation, and rain forecasts.
2. **Cycle & Soak Scheduling**: Eliminates runoff on heavy soils or slopes by pulsing water delivery with soak periods.
3. **Garden Bed & Crop Tracking**: Interactive 2D bed layout, crop growth stages, days-to-maturity countdowns, and companion planting matrix.
4. **Ecosystem Interoperability**: Seamlessly ties into `SoilSense-io`, `WaterTankControl-io`, `RainHarvest-io`, and `SolarControl-io`.
5. **Home Assistant Ingress & Native Entities**: Direct sidebar dashboard and standard entity auto-discovery via MQTT / Supervisor API.
