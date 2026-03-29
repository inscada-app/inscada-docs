---
title: "Rotate & Move"
description: "Rotation and translation animations"
sidebar:
  order: 13
---

## Rotate (Rotation)

**Rotate** rotates an SVG element based on a value. It is used for circular displays such as gauge needles, valve positions, wind direction, and compass.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Rotate |
| **Suitable SVG Elements** | `<g>`, `<path>`, `<line>`, `<rect>` |
| **Expression Type** | Tag, Expression |

### Configuration (Props)

| Property | Description |
|----------|-------------|
| **cx** | Rotation center X coordinate |
| **cy** | Rotation center Y coordinate |
| **minAngle** | Minimum angle (degrees) |
| **maxAngle** | Maximum angle (degrees) |
| **minValue** | Minimum value |
| **maxValue** | Maximum value |

### SVG Preparation — Analog Gauge

```xml
<svg viewBox="0 0 200 200">
  <!-- Dial background -->
  <circle cx="100" cy="100" r="90" fill="none" stroke="#ddd" stroke-width="4"/>

  <!-- Dial markings -->
  <line x1="100" y1="15" x2="100" y2="25" stroke="#333" stroke-width="2"/>

  <!-- Needle -->
  <g id="gauge_needle">
    <line x1="100" y1="100" x2="100" y2="20"
          stroke="#e74c3c" stroke-width="3" stroke-linecap="round"/>
    <circle cx="100" cy="100" r="5" fill="#e74c3c"/>
  </g>

  <!-- Value text -->
  <text id="gauge_value" x="100" y="160"
        text-anchor="middle" font-size="18" fill="#333">--</text>
</svg>
```

### Configuration

- `gauge_needle` → **Rotate**
  - Props: cx=100, cy=100, minAngle=-135, maxAngle=135, minValue=0, maxValue=100
  - Expression Type: Tag → `Temperature_C`
- `gauge_value` → **Get**
  - Expression: `ins.getVariableValue("Temperature_C").value.toFixed(1) + " °C"`

Result: At 0°C the needle is at the lower left (-135°), at 100°C at the lower right (135°).

### Valve Position Example

```javascript
// 0 = closed (0°), 100 = open (90°)
var pos = ins.getVariableValue("Valve_Position").value;
return pos * 0.9; // 0-90 degrees
```

---

## Move (Translation)

**Move** translates an SVG element along the X and/or Y axis based on a value. It is used for linear motion animations such as level indicators, conveyor positions, and elevators.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Move |
| **Suitable SVG Elements** | `<g>`, `<rect>`, `<circle>`, `<image>` |
| **Expression Type** | Tag, Expression |

### Configuration (Props)

| Property | Description |
|----------|-------------|
| **axis** | Movement axis: `x` or `y` |
| **minPos** | Minimum position (pixels) |
| **maxPos** | Maximum position (pixels) |
| **minValue** | Minimum value |
| **maxValue** | Maximum value |

### Elevator Example

```xml
<svg viewBox="0 0 100 400">
  <!-- Shaft -->
  <rect x="20" y="10" width="60" height="380" fill="none" stroke="#999"/>
  <!-- Cabin -->
  <g id="elevator_cabin">
    <rect x="25" y="0" width="50" height="40" fill="#3498db" rx="3"/>
  </g>
</svg>
```

- `elevator_cabin` → **Move**
  - Props: axis=y, minPos=10, maxPos=350, minValue=0, maxValue=10
  - Expression Type: Tag → `Floor_Position`
  - At floor 0 the cabin is at the bottom, at floor 10 at the top

### Level Indicator Example

```javascript
// Tank level 0-100% → Y position 200-0 (reverse direction)
var level = ins.getVariableValue("Tank_Level").value;
return 200 - (level * 2);
```
