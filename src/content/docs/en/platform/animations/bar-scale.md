---
title: "Bar & Scale"
description: "Bar gauge and scaling animations"
sidebar:
  order: 12
---

## Bar (Bar Gauge)

**Bar** changes the height or width of an SVG rectangle proportionally to a value. It is used for displays such as tank level, progress bar, and load indicator.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Bar |
| **Suitable SVG Elements** | `<rect>` |
| **Expression Type** | Tag, Expression |

### Configuration (Props)

| Property | Description |
|----------|-------------|
| **direction** | Direction: `vertical` (bottom to top) or `horizontal` (left to right) |
| **min** | Minimum value (bar is empty at this value) |
| **max** | Maximum value (bar is full at this value) |
| **fillColor** | Fill color |

### SVG Preparation — Vertical Bar

```xml
<!-- Tank frame -->
<rect x="40" y="20" width="40" height="200" fill="none" stroke="#666" stroke-width="2"/>
<!-- Level bar (initially empty) -->
<rect id="tank_level" x="42" y="220" width="36" height="0" fill="#3498db"/>
```

### Expression Examples

**Tag:**
```
Temperature_C
```
Props: `min=0, max=100, direction=vertical`
→ At 50°C the bar is 50% full

**Expression with color change:**
```javascript
var val = ins.getVariableValue("Temperature_C").value;
// Return the value, color can be set separately with a Color element
return val;
```

### Full Example — Horizontal Progress Bar

```xml
<svg viewBox="0 0 400 60">
  <!-- Background -->
  <rect x="10" y="20" width="300" height="20" fill="#e0e0e0" rx="3"/>
  <!-- Progress -->
  <rect id="progress_bar" x="10" y="20" width="0" height="20" fill="#2196F3" rx="3"/>
  <!-- Percentage text -->
  <text id="progress_text" x="320" y="35" font-size="14">0%</text>
</svg>
```

- `progress_bar` → Bar, Tag: `PowerFactor`, Props: min=0, max=1, horizontal
- `progress_text` → Get, Expression: `(ins.getVariableValue("PowerFactor").value * 100).toFixed(0) + "%"`

---

## Scale (Scaling)

**Scale** enlarges or shrinks an SVG element or group based on a value. It uses the `transform: scale()` CSS property.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Scale |
| **Suitable SVG Elements** | `<g>`, `<rect>`, `<circle>`, `<path>` |
| **Expression Type** | Tag, Expression |

### Expression Examples

```javascript
// Convert 0-1000 kW range to 0.5-2.0 scale
var power = ins.getVariableValue("ActivePower_kW").value;
return 0.5 + (power / 1000) * 1.5;
```

### Usage Scenarios

- Symbol that grows/shrinks based on production quantity
- Dynamically sized gauge needle
- Warning icon that enlarges in alarm state
