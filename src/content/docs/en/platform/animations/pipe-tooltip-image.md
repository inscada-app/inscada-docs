---
title: "Pipe, Tooltip & Image"
description: "Flow animation, information tooltip, and dynamic image switching"
sidebar:
  order: 15
---

## Pipe (Flow Animation)

**Pipe** creates an animation showing the flow direction in pipelines or cables. It visualizes liquid/gas flow with a moving dash pattern on the SVG line.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Pipe |
| **Suitable SVG Elements** | `<path>`, `<line>`, `<polyline>` |
| **Expression Type** | Tag (Boolean), Expression |

### SVG Preparation

```xml
<!-- Pipeline -->
<path id="main_pipe" d="M 50,100 L 200,100 L 200,250 L 350,250"
      fill="none" stroke="#3498db" stroke-width="6"
      stroke-dasharray="15,10"/>
```

### How It Works

- `true` → `stroke-dashoffset` CSS animation starts, dashes move
- `false` → animation stops, line remains static

### Expression Examples

**Boolean — Tag:**
```
Pump_Running
```
When the pump is running, flow is visible in the pipeline.

**Conditional — Expression:**
```javascript
var flow = ins.getVariableValue("Flow_Rate").value;
return flow > 0; // Animation when there is flow
```

---

## Tooltip (Information Balloon)

**Tooltip** is a popup balloon that shows detail information when hovering over an SVG element. It provides additional information without taking up space on the main screen.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Tooltip |
| **Suitable SVG Elements** | All |
| **Expression Type** | Expression, Text |

### Expression Examples

**Static text — Text:**
```
Active Power Measurement - Transformer output
```

**Dynamic — Expression (supports HTML):**
```javascript
var p = ins.getVariableValue("ActivePower_kW");
var v = ins.getVariableValue("Voltage_V");
return "<b>Energy Analyzer</b><br>"
     + "Power: " + p.value.toFixed(1) + " kW<br>"
     + "Voltage: " + v.value.toFixed(1) + " V<br>"
     + "Last update: " + new Date(p.dateInMs).toLocaleTimeString();
```

When the user hovers over the SVG element, a rich HTML tooltip appears.

---

## Image (Image Switching)

**Image** changes the source (`href`) of an SVG `<image>` element based on a value. It is used for showing different icons, photos, or symbols based on state.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Image |
| **Suitable SVG Elements** | `<image>`, `<rect>` (with foreignObject) |
| **Expression Type** | Switch, Expression |

### SVG Preparation

```xml
<image id="equipment_icon" x="50" y="50" width="64" height="64"
       href="/images/default.png"/>
```

### Switch Example

```
0 → /images/motor-off.png
1 → /images/motor-on.png
2 → /images/motor-fault.png
3 → /images/motor-maintenance.png
```

### Expression Example

```javascript
var status = ins.getVariableValue("Motor_Status").value;
var base = "/images/motor-";
if (status === 0) return base + "off.png";
if (status === 1) return base + "on.png";
if (status === 2) return base + "fault.png";
return base + "unknown.png";
```

## AlarmIndication (Alarm Indicator)

**AlarmIndication** automatically displays the status of an alarm group with color and blinking. It uses the color settings (OnNoAck, OnAck, OffNoAck, OffAck) from the alarm group definition.

### Usage

| Field | Value |
|-------|-------|
| **Type** | AlarmIndication |
| **Suitable SVG Elements** | `<rect>`, `<circle>`, `<path>` |
| **Expression Type** | Alarm |
| **Expression** | Alarm group reference |

### Automatic Behavior

| Alarm State | Appearance |
|-------------|------------|
| Normal | OffAck color from the alarm group |
| Fired + No Ack | OnNoAck color, blinking |
| Fired + Ack | OnAck color, solid |
| Off + No Ack | OffNoAck color |

You do not need to set the colors manually — the colors from the alarm group definition are applied automatically.
