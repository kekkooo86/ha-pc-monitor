# ha-pc-monitor — Copilot Instructions

## Project Overview

Home Assistant package that receives system metrics from a PC via MQTT (published by sysmon-mqtt) and provides:
- MQTT sensor definitions (package YAML)
- Blueprint automation for RGB LED color based on CPU temperature
- Lovelace dashboard for PC monitoring

**Companion project:** [sysmon-mqtt](https://github.com/kekkooo86/sysmon-mqtt)

## Structure

```
blueprints/automation/cpu_temp_led_color.yaml   # HA blueprint (parametrized)
packages/pc_monitor.yaml                         # MQTT sensors package
lovelace/pc_monitor_dashboard.yaml               # Dashboard view
README.md
```

## Key Conventions

- Topic format: `{prefix}/sensor/{sensor_name}/state` — prefix matches sysmon-mqtt setting
- Default prefix: `homeassistant`
- No `input_number` helper needed — blueprint triggers directly on sensor state
- Dashboard uses `custom:grid-layout` via `layout-card`
- Sensor unique_ids follow pattern: `pc_monitor_{sensor_name}`
