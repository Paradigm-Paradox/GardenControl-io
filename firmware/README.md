# GardenControl-io: ESP32 Firmware Configurations

This directory contains pre-configured, modular firmware templates for building **ESP32** hardware nodes that interface seamlessly with **GardenControl-io** and **Home Assistant**.

## Available Recipes (`esphome/`)

1. **`valve_manifold_eth_poe.yaml`**:
   - **Board**: Olimex ESP32-POE / WT32-ETH01
   - **Communication**: Hardwired Ethernet (LAN8720A / 802.3af PoE)
   - **Features**: 4-8 solenoid valve relays, master pump interlock, pulse flow meter counter (liters & flow rate), analog water pressure transducer, and hardware runtime safety interlocks.

2. **`garden_bed_wifi_sensor.yaml`**:
   - **Board**: ESP32-C3 / ESP32-S3 / ESP32 NodeMCU
   - **Communication**: 2.4 GHz Wi-Fi with Deep Sleep
   - **Features**: Switched-VCC capacitive soil moisture (VWC %), DS18B20 waterproof soil temperature, BME280 air microclimate, battery voltage monitoring. Designed for solar/18650 LiFePO4 battery power.

3. **`ble_soil_tracker.yaml`**:
   - **Board**: Standard ESP32-WROOM-32
   - **Communication**: Wi-Fi + Bluetooth Low Energy (BLE)
   - **Features**: Discovers and ingests broadcasts from Bluetooth wireless soil sensors (such as Xiaomi Flower Care / HHCC Flora, SoilSense, Inkbird) and proxies them to Home Assistant.

## Flashing Instructions
Install the [ESPHome Dashboard](https://esphome.io/) in Home Assistant or compile via CLI:
```bash
esphome run firmware/esphome/valve_manifold_eth_poe.yaml
```
