# ha-pc-monitor

Home Assistant package that receives system metrics from a PC via MQTT and displays them on a configurable dashboard. Includes a blueprint automation that changes the color of an RGB light based on CPU temperature.

Data is published to MQTT by **[sysmon-mqtt](https://github.com/kekkooo86/sysmon-mqtt)**, a companion Electron app that runs on the monitored PC.

---

## What's included

| File | Description |
|---|---|
| `packages/pc_monitor.yaml` | MQTT sensors: CPU temp/load, RAM, GPU, disk, network |
| `blueprints/automation/cpu_temp_led_color.yaml` | Blueprint: RGB light color follows a selected PC metric |
| `lovelace/pc_monitor_dashboard.yaml` | Lovelace view using `custom:grid-layout` for a responsive PC dashboard |

---

## Requirements

- Home Assistant (any recent version)
- An MQTT broker (e.g. Mosquitto add-on)
- **[sysmon-mqtt](https://github.com/kekkooo86/sysmon-mqtt)** running on the PC you want to monitor
- `layout-card` installed in Home Assistant for the Lovelace view (`custom:grid-layout`)

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

Then in HA go to **Settings → Automations → Blueprints** and create a new automation from `PC Metric LED Color`.

**Blueprint inputs:**

| Input | Description | Default |
|---|---|---|
| Metric Sensor | Any numeric sysmon-mqtt sensor, such as `sensor.pc_cpu_temperature` or `sensor.pc_cpu_usage` | — |
| RGB Light | Your LED light entity | — |
| Maximum metric value | Absolute maximum expected for the sensor | `90` |
| Low threshold percentage | Percentage of max where the LED reaches the 2nd gauge color | `33.3%` |
| Mid threshold percentage | Percentage of max where the LED reaches the 3rd gauge color | `63.3%` |
| High threshold percentage | Percentage of max where the LED reaches the 4th gauge color | `83.3%` |
| Cold color | 1st gauge color anchor | `[33, 150, 243]` |
| Warm color | 2nd gauge color anchor | `[76, 175, 80]` |
| Hot color | 3rd gauge color anchor | `[255, 152, 0]` |
| Critical color | 4th gauge color anchor | `[244, 67, 54]` |

### 4. Add the dashboard layout

The included view uses `type: custom:grid-layout`, so you need the **layout-card** custom card first.

#### Install layout-card

Recommended via HACS:

1. Open **HACS → Frontend**
2. Search for `layout-card`
3. Install it
4. Reload Home Assistant

If you do not use HACS, install it manually as a Lovelace resource following the project instructions:
https://github.com/thomasloven/lovelace-layout-card

#### Add the view

Create a new view in your dashboard using the **Manual** option, then paste the contents of `lovelace/pc_monitor_dashboard.yaml`.

> This file is a single Lovelace **view layout**, not a full dashboard export. Paste it as the configuration of one view.

> **Note:** Entity IDs in the layout assume the default sensor names from `packages/pc_monitor.yaml`. If your entities differ, update them in the YAML before saving.

---

## Topic reference

All topics follow the pattern `{prefix}/sensor/{name}/state`.

| Sensor | Topic (default prefix) |
|---|---|
| CPU Temperature | `homeassistant/sensor/cpu_temp_max/state` |
| CPU Usage | `homeassistant/sensor/cpu_usage/state` |
| RAM Used % | `homeassistant/sensor/ram_used_percent/state` |
| RAM Used GB | `homeassistant/sensor/ram_used_gb/state` |
| GPU Temperature | `homeassistant/sensor/gpu_temp_edge/state` |
| GPU Usage | `homeassistant/sensor/gpu_usage/state` |
| GPU VRAM Used % | `homeassistant/sensor/gpu_vram_used_percent/state` |
| Disk Used % (root) | `homeassistant/sensor/disk_root_use_percent/state` |
| Disk Free GB (root) | `homeassistant/sensor/disk_root_free_gb/state` |
| Network Download | `homeassistant/sensor/net_<iface>_rx_kbs/state` (e.g. `net_eno1_rx_kbs`) |
| Network Upload | `homeassistant/sensor/net_<iface>_tx_kbs/state` (e.g. `net_eno1_tx_kbs`) |

---

## LED color logic

| Metric range | Color |
|---|---|
| `0 → low threshold` | Cold → warm (interpolated) |
| `low → mid` | Warm → hot (interpolated) |
| `mid → high` | Hot → critical (interpolated) |
| `≥ high threshold` | Critical color |

The blueprint calculates absolute thresholds from the configured maximum value:

```text
absolute threshold = maximum metric value × percentage / 100
```

With the default values, a maximum of `90` produces thresholds at about
`30`, `57`, and `75`, matching the included CPU gauge.

For percentage-based sensors such as CPU usage, set the maximum value to `100`.

Threshold percentages and the 4 RGB anchor colors are configurable per automation.
The defaults are aligned with the CPU gauge shipped in `lovelace/pc_monitor_dashboard.yaml`:
blue `#2196F3`, green `#4CAF50`, orange `#FF9800`, red `#F44336`.
This works with any numeric metric exposed by the package, including temperature
and percentage-based usage sensors.

---

## License

MIT
