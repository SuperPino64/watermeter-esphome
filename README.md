# Muino Water Meter ESPHome (Actaris Aquadis+ Edition)

## Overview

This fork contains practical improvements focused on:

- Actaris Aquadis+ compatibility
- Improved calibration stability
- Faster debugging workflow
- Home Assistant integration
- Reduced Home Assistant database growth
- Phase transition diagnostics

---

## Tested Water Meter

Successfully tested with:

- Actaris Aquadis+

The sensor positioning, phase detection and flow calculations have been validated on this meter type.

---

## Major Changes

### Removed 120° Mode

The original firmware contained support for 120° and 180° spindle detection.

This fork uses:

- 180° mode only

Benefits:

- Simpler code path
- Less risk of incorrect phase detection
- Easier troubleshooting

---

### Speed Mode Always Active

Speed Mode is now intended as the standard operating mode.

Benefits:

- Faster response to flow changes
- More accurate phase tracking
- Better live debugging

---

### Debug JSON Removed

The original debug_JSON telemetry has been removed.

Reasons:

- Reduced network traffic
- Reduced Home Assistant database growth
- Improved readability

---

## Phase Diagnostics

Additional diagnostics have been added.

Every phase transition can be logged:

```cpp
ESP_LOGI(
    "phase_trace",
    "phase %d -> %d",
    phase_before,
    phase_after);
```

Unexpected transitions generate warnings:

```cpp
ESP_LOGW(
    "phase_check",
    "Unexpected phase jump %d -> %d",
    phase_before,
    phase_after);
```

Examples:

```text
phase 0 -> 1
phase 1 -> 2
phase 2 -> 3
phase 3 -> 4
phase 4 -> 5
phase 5 -> 6
```

Warning example:

```text
Unexpected phase jump 1 -> 4
```

---

## Improved Startup Calibration

The original firmware initialized:

```cpp
max_a = 0;
max_b = 0;
max_c = 0;
```

This fork initializes with measured values.

Benefits:

- Faster convergence
- Fewer startup spikes
- Better automatic calibration

History buffers are also initialized immediately.

---

## Home Assistant Integration

### Debug Sensors

The following sensors are available:

- sensa_sub
- sensb_sub
- sensc_sub
- Phase

To reduce database growth:

```yaml
update_interval: never
force_update: false
```

The sensors are only updated when Debug Mode is enabled.

---

## ApexCharts Debug Graph

### A/B/C Sensors

```yaml
type: custom:apexcharts-card
graph_span: 2min
update_interval: 0.5s
apex_config:
  chart:
    height: 350
  stroke:
    width: 1
    curve: straight
  legend:
    show: true
series:
  - entity: sensor.meterkast_muino_water_meter_sensa_sub
    name: A
    type: line
  - entity: sensor.meterkast_muino_water_meter_sensb_sub
    name: B
    type: line
  - entity: sensor.meterkast_muino_water_meter_sensc_sub
    name: C
    type: line
```

### Phase Graph

```yaml
type: custom:apexcharts-card
graph_span: 2min
update_interval: 0.5s

apex_config:
  chart:
    height: 350

series:
  - entity: sensor.meterkast_muino_water_meter_phase
    name: Phase
```

---

## Utility Meter in Home Assistant

Recommended setup:

Create a template sensor with offset:

```yaml
template:
  - sensor:
      - name: Watermeter Totaal
        unit_of_measurement: "L"
        device_class: water
        state_class: total_increasing
        state: >
          {{
            (states('sensor.meterkast_muino_water_meter_liters')|float(0)
            + 2388.426) | round(3)
          }}
```

Create Utility Meters based on:

```text
sensor.watermeter_totaal
```

---

## Flow Calculation

Current implementation:

```cpp
float flow_rate = 10000.0f / float(time_diff);

if (flow_rate > 50.0f)
    flow_rate = 50.0f;
```

Benefits:

- Limits unrealistic spikes
- Stable Home Assistant graphs

---

## Recommended Alignment Procedure

1. Enable Debug Mode.
2. Open ApexCharts graph.
3. Run a constant water flow.
4. Adjust the sensor position.
5. Aim for smooth A/B/C rotation.
6. Verify phase progression.
7. Confirm liter counting.

Expected phase sequence:

```text
0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 0
```

---




Environment:

- ESPHome 2026.10.x
- Home Assistant
- ESP32-C3 (Seeed XIAO)
- Actaris Aquadis+
