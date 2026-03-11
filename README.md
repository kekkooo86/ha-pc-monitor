# ha-pc-monitor

Home Assistant package that receives system metrics from a PC via MQTT and displays them on a configurable dashboard. Includes a blueprint automation that changes the color of an RGB light based on CPU temperature.

Data is published to MQTT by **[sysmon-mqtt](https://github.com/kekkooo86/sysmon-mqtt)**, a companion Electron app that runs on the monitored PC.

---

## What's included

| File | Description |
|---|---|
| `packages/pc_monitor.yaml` | MQTT sensors: CPU temp/load, RAM, GPU, disk, network |
| `blueprints/automation/cpu_temp_led_color.yaml` | Blueprint: RGB light color follows CPU temperature |
| `lovelace/pc_monitor_dashboard.yaml` | Lovelace dashboard view with gauges and history graphs |

---

## Requirements

- Home Assistant (any recent version)
- An MQTT broker (e.g. Mosquitto add-on)
- **[sysmon-mqtt](https://github.com/kekkooo86/sysmon-mqtt)** running on the PC you want to monitor

---

## Data flow

```
PC (sysmon-mqtt)
  └─► MQTT broker
        └─► Home Assistant
              ├─► Sensors (packages/pc_monitor.yaml)
              ├─► Dashboard (lovelace/)
              └─► LED automation (blueprints/)
```

---

## Installation

### 1. Configure sysmon-mqtt

Install and run **[sysmon-mqtt](https://github.com/kekkooo86/sysmon-mqtt)** on your PC.

In the Settings window, set the **Topic Prefix** (default: `homeassistant`). All sensors will publish to:
```
{prefix}/sensor/{sensor_name}/state
```

### 2. Add the package

Copy `packages/pc_monitor.yaml` into your HA config directory under `packages/`:

```
config/
└── packages/
    └── pc_monitor.yaml
```

Then enable packages in `configuration.yaml` (add only if not already present):

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Open `packages/pc_monitor.yaml` and update the `state_topic` values if you changed the Topic Prefix in sysmon-mqtt. Replace `homeassistant` with your prefix in every `state_topic` line.

Restart Home Assistant.

### 3. Install the blueprint

#### Option A — one-click import

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/kekkooo86/ha-pc-monitor/blob/main/blueprints/automation/cpu_temp_led_color.yaml)

#### Option B — manual

Copy `blueprints/automation/cpu_temp_led_color.yaml` to:
```
config/blueprints/automation/ha-pc-monitor/cpu_temp_led_color.yaml
```

Then in HA go to **Settings → Automations → Blueprints** and create a new automation from `CPU Temperature LED Color`.

**Blueprint inputs:**

| Input | Description | Default |
|---|---|---|
| CPU Temperature Sensor | The `sensor.pc_cpu_temperature` entity | — |
| RGB Light | Your LED light entity | — |
| Cold threshold | Below this → blue | 30°C |
| Mid threshold | Blue→green transition point | 57°C |
| Hot threshold | Above this → red | 85°C |

### 4. Add the dashboard

In HA go to **Settings → Dashboards → Add dashboard** (or edit an existing one in raw YAML mode) and paste the contents of `lovelace/pc_monitor_dashboard.yaml`.

> **Note:** Entity IDs in the dashboard file assume the default sensor names from `packages/pc_monitor.yaml`. If you rename sensors, update the entity IDs accordingly.

---

## Topic reference

All topics follow the pattern `{prefix}/sensor/{name}/state`.

| Sensor | Topic (default prefix) |
|---|---|
| CPU Temperature | `homeassistant/sensor/cpu_temp_max/state` |
| CPU Load | `homeassistant/sensor/cpu_load/state` |
| RAM Used % | `homeassistant/sensor/ram_used_pct/state` |
| RAM Used GB | `homeassistant/sensor/ram_used_gb/state` |
| GPU Temperature | `homeassistant/sensor/gpu_temp/state` |
| GPU Load | `homeassistant/sensor/gpu_load/state` |
| GPU VRAM Used % | `homeassistant/sensor/gpu_vram_used_pct/state` |
| Disk Used % (root) | `homeassistant/sensor/disk_used_pct_root/state` |
| Disk Free GB (root) | `homeassistant/sensor/disk_free_gb_root/state` |
| Network Download | `homeassistant/sensor/net_download_{iface}/state` |
| Network Upload | `homeassistant/sensor/net_upload_{iface}/state` |

---

## LED color logic

| Temperature range | Color |
|---|---|
| ≤ cold threshold | 🔵 Blue `[0, 0, 255]` |
| cold → mid | 🔵→🟢 Blue to green (interpolated) |
| mid → hot | 🟢→🔴 Green to red (interpolated) |
| ≥ hot threshold | 🔴 Red `[255, 0, 0]` |

Thresholds are configurable per-automation when creating from the blueprint.

---

## License

MIT
