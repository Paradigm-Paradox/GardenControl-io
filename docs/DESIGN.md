# GardenControl-io: Functional Design & Specification

## 1. Overview
**GardenControl-io** is an intelligent garden and raised-bed automation platform engineered as a native Home Assistant Add-on. It unifies microclimate monitoring, biological crop tracking, companion planting intelligence, and smart adaptive irrigation into a unified local control hub.

---

## 2. Core Functional Modules

### 2.1 Zone & Garden Bed Architecture
- **Garden Beds**: Distinct physical growing areas (raised beds, in-ground rows, wicking beds, container pots, orchard plots).
- **Irrigation Zones**: Physical valve/circuit lines assigned to one or more beds.
- **Soil Characteristics**:
  - Soil Type: Sandy, Loamy, Clay, Peat, Compost-rich.
  - Infiltration Rate ($mm/hr$): Dictates maximum pulse duration before runoff occurs.
  - Field Capacity ($FC$) and Permanent Wilting Point ($PWP$).
  - Allowable Depletion Fraction ($p$): Typically 0.40 - 0.50 before stress occurs.

### 2.2 Biological Crop Tracker
- **Crop Registry**:
  - Growth Stages: `Germination`, `Vegetative`, `Flowering/Fruiting`, `Ripening/Harvest`.
  - Stage-specific Crop Coefficients ($K_c$):
    - *Example (Tomato)*: Initial = 0.6, Mid-season = 1.15, Late = 0.8.
  - Root Depth ($Z_r$): Dynamic over time from seeding to full maturity.
  - Companion Planting Rules: Synergistic pairs (e.g. Tomato + Basil + Marigold) and antagonist warnings (e.g. Beans + Onions, Tomato + Fennel).
  - Pest & Disease Treatment Log.

### 2.3 Smart Adaptive Irrigation Engine
1. **Evapotranspiration ($ET_0$) Calculation**:
   - Primary: Hargreaves-Samani method (using daily max/min temperature and extraterrestrial solar radiation $R_a$).
   - Enhanced: Penman-Monteith equation when solar radiation, wind speed, and relative humidity sensors are available.
2. **Crop Water Need**:
   $$ET_c = K_c \times ET_0 - P_{eff}$$
   where $P_{eff}$ is effective rainfall measured by local rain gauges or HA weather integration.
3. **Cycle and Soak Execution**:
   - To prevent surface pooling on clay/compacted soils, total watering run is divided into multiple short bursts:
     - `Pulse Duration`: e.g. 8 minutes
     - `Soak Duration`: e.g. 15 minutes
     - `Cycle Count`: Total target volume / pulse capacity
4. **Safety & Hardware Interlocks**:
   - **Max Run Duration Watchdog**: Hard timeout to prevent stuck solenoid valves from flooding beds.
   - **Water Source Verification**: Handshake with `WaterTankControl-io` or cistern sensor: if reservoir < 15%, suspend non-critical zones.
   - **Concurrency Limiting**: Only 1 or 2 valves active simultaneously to maintain constant line pressure.
   - **Solar Coordination**: Option to schedule pumping cycles during peak PV production hours reported by `SolarControl-io`.

---

## 3. Database Schema (SQLite: `/data/gardencontrol.db`)

```sql
-- Garden Zones (Valves & Solenoids)
CREATE TABLE IF NOT EXISTS zones (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    valve_entity TEXT NOT NULL,
    flow_rate_lpm REAL DEFAULT 10.0,
    max_duration_minutes INTEGER DEFAULT 45,
    is_enabled INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Physical Beds
CREATE TABLE IF NOT EXISTS beds (
    id TEXT PRIMARY KEY,
    zone_id TEXT REFERENCES zones(id),
    name TEXT NOT NULL,
    bed_type TEXT DEFAULT 'raised_bed', -- raised_bed, in_ground, container, wicking
    width_cm REAL DEFAULT 120.0,
    length_cm REAL DEFAULT 240.0,
    depth_cm REAL DEFAULT 30.0,
    soil_type TEXT DEFAULT 'loam',
    soil_sensor_entity TEXT,
    moisture_target_pct REAL DEFAULT 45.0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Crop Species Catalog
CREATE TABLE IF NOT EXISTS crop_catalog (
    id TEXT PRIMARY KEY,
    common_name TEXT NOT NULL,
    botanical_name TEXT,
    category TEXT, -- vegetable, herb, fruit, flower
    days_to_germinate INTEGER,
    days_to_maturity INTEGER,
    sun_exposure TEXT DEFAULT 'full_sun', -- full_sun, partial_shade, shade
    water_demand TEXT DEFAULT 'medium', -- low, medium, high
    kc_initial REAL DEFAULT 0.5,
    kc_mid REAL DEFAULT 1.0,
    kc_late REAL DEFAULT 0.8,
    root_depth_cm REAL DEFAULT 30.0,
    spacing_cm REAL DEFAULT 30.0
);

-- Planted Instances
CREATE TABLE IF NOT EXISTS plantings (
    id TEXT PRIMARY KEY,
    bed_id TEXT NOT NULL REFERENCES beds(id),
    crop_id TEXT NOT NULL REFERENCES crop_catalog(id),
    x_cm REAL DEFAULT 0.0,
    y_cm REAL DEFAULT 0.0,
    quantity INTEGER DEFAULT 1,
    planted_at DATE NOT NULL,
    current_stage TEXT DEFAULT 'vegetative',
    expected_harvest DATE,
    actual_harvest DATE,
    notes TEXT
);

-- Irrigation Logs & Telemetry
CREATE TABLE IF NOT EXISTS irrigation_runs (
    id TEXT PRIMARY KEY,
    zone_id TEXT NOT NULL REFERENCES zones(id),
    trigger_source TEXT NOT NULL, -- schedule, moisture_deficit, manual, weather_override
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    duration_seconds INTEGER,
    estimated_liters REAL,
    status TEXT DEFAULT 'completed' -- completed, interrupted, aborted_safety
);
```

---

## 4. REST API Specification

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/api/status` | GET | Add-on operational health, active irrigation runs, safety interlocks |
| `/api/zones` | GET, POST | List all irrigation zones / Create new zone |
| `/api/zones/{id}` | GET, PUT, DELETE | Manage specific zone config & entity linkage |
| `/api/zones/{id}/run` | POST | Manually trigger watering run with optional cycle-soak override |
| `/api/zones/{id}/stop` | POST | Immediate safety halt for zone |
| `/api/beds` | GET, POST | Manage garden beds, physical dimensions, and assigned sensors |
| `/api/plantings` | GET, POST, DELETE | Track planted crops per bed, location, and growth stage |
| `/api/crops` | GET, POST | Access crop catalog and companion planting recommendations |
| `/api/weather/et` | GET | Current calculated $ET_0$, 7-day water budget, and rain forecasts |
| `/api/history/runs` | GET | Historical water usage, runtime logs, and moisture trends |

---

## 5. Home Assistant Integration

### 5.1 MQTT Auto-Discovery
GardenControl-io automatically announces the following entities to Home Assistant:
- `valve.garden_zone_{id}`: Direct control valve with remaining run-time attribute.
- `sensor.garden_zone_{id}_moisture`: Aggregated soil moisture percentage.
- `sensor.garden_zone_{id}_water_today`: Total liters applied today.
- `sensor.garden_et0_daily`: Current reference evapotranspiration ($mm$).
- `switch.garden_weather_delay`: Manual/automated weather pause toggle.

### 5.2 Supervisor Ingress UI
- Served over internal HTTP port `8099`.
- Embedded into Home Assistant sidebar via `ingress: true`.
- Responsive for mobile (Home Assistant Companion App) and desktop screens.
