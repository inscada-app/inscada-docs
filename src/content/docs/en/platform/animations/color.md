---
title: "Color"
description: "Changing color based on value"
sidebar:
  order: 11
---

**Color** dynamically changes the fill or stroke color of an SVG element based on a variable value. It is used for status indicators, alarm coloring, and threshold-based visualization.

## Usage

| Field | Value |
|-------|-------|
| **Type** | Color |
| **Suitable SVG Elements** | `<rect>`, `<circle>`, `<ellipse>`, `<polygon>`, `<path>`, `<text>` |
| **Expression Type** | Expression, Switch, Tetra Color |

## SVG Preparation

```xml
<circle id="status_led" cx="50" cy="50" r="12" fill="#cccccc" stroke="#999"/>
```

## Configuration Examples

### Boolean Status — With Switch

Expression Type: **Switch**
```
true → #00cc00
false → #ff0000
```
Green if the motor is running, red if stopped.

### Multiple States — With Switch

```
0 → #999999
1 → #00cc00
2 → #ff0000
3 → #ff8800
```
0=Off (gray), 1=Running (green), 2=Fault (red), 3=Warning (orange)

### Threshold-Based — With Expression

Expression Type: **Expression**
```javascript
var temp = ins.getVariableValue("Temperature_C").value;
if (temp > 80) return "#ff0000";      // red — critical
if (temp > 60) return "#ff8800";      // orange — warning
if (temp > 40) return "#ffcc00";      // yellow — caution
return "#00cc00";                     // green — normal
```

### Gradient / Blinking

Blinking effect between two colors:
```javascript
return "#ff0000/#ffffff";  // red ↔ white blinking
```

When two colors are specified with the `/` character, an SVG `<animate>` element is created and a color transition is performed.

### Tetra Color (Alarm 4 Colors)

Expression Type: **Tetra Color**

Automatic coloring based on the four states of an alarm group:

| State | Default Color |
|-------|--------------|
| Fired + No Ack | Red blinking |
| Fired + Ack | Red solid |
| Off + No Ack | Yellow |
| Off + Ack | Normal (gray/white) |

Automatically applies the color settings from the alarm group definition.

## Full SVG Example

```xml
<svg viewBox="0 0 300 100">
  <!-- 3 motor status indicators -->
  <g transform="translate(30,50)">
    <circle id="motor1_led" r="15" fill="#ccc"/>
    <text y="35" text-anchor="middle" font-size="11">Motor 1</text>
  </g>
  <g transform="translate(150,50)">
    <circle id="motor2_led" r="15" fill="#ccc"/>
    <text y="35" text-anchor="middle" font-size="11">Motor 2</text>
  </g>
  <g transform="translate(270,50)">
    <circle id="motor3_led" r="15" fill="#ccc"/>
    <text y="35" text-anchor="middle" font-size="11">Motor 3</text>
  </g>
</svg>
```

For each `motor*_led`, a Color element:
- Expression Type: Switch
- Expression: `Motor1_Status` (Tag)
- Switch: `true → #00cc00 | false → #ff0000`
