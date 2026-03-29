---
title: "Opacity, Visibility & Blink"
description: "Opacity, show/hide, and blinking animations"
sidebar:
  order: 14
---

## Opacity

**Opacity** adjusts the transparency of an SVG element based on a value. It is used for effects such as fading a device symbol based on communication status or zone brightness based on power state.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Opacity |
| **Suitable SVG Elements** | All |
| **Expression Type** | Tag, Expression |

### Expression Examples

```javascript
// Based on connection status: connected=opaque, disconnected=faded
var status = ins.getConnectionStatus("MODBUS-PLC");
return status === "Connected" ? 1.0 : 0.3;
```

```javascript
// Proportional opacity based on value (0-100 → 0.2-1.0)
var val = ins.getVariableValue("Signal_Strength").value;
return 0.2 + (val / 100) * 0.8;
```

---

## Visibility (Show/Hide)

**Visibility** shows or completely hides an SVG element based on a condition (`display: none`). Unlike Opacity: it is not gradual — it is either fully visible or fully hidden.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Visibility |
| **Suitable SVG Elements** | All (especially `<g>` groups) |
| **Expression Type** | Tag (Boolean), Expression |

### SVG Preparation

```xml
<!-- Alarm icon — hidden by default -->
<g id="alarm_warning" display="none">
  <polygon points="100,10 120,50 80,50" fill="#ff0000"/>
  <text x="100" y="43" text-anchor="middle" fill="white" font-size="20">!</text>
</g>
```

### Expression Examples

**Boolean variable — Tag:**
```
GridStatus
```
`true` → visible, `false` → hidden

**Threshold condition — Expression:**
```javascript
var temp = ins.getVariableValue("Temperature_C").value;
return temp > 70; // Show warning icon above 70°C
```

**Multiple conditions:**
```javascript
var power = ins.getVariableValue("ActivePower_kW").value;
var status = ins.getVariableValue("GridStatus").value;
return power > 500 && status; // both high power and connected
```

### Usage Scenario — Alarm/Normal Icon Switching

```xml
<!-- Normal state icon -->
<g id="icon_normal">
  <circle r="15" fill="#00cc00"/>
  <text text-anchor="middle" y="5" fill="white">✓</text>
</g>

<!-- Alarm state icon -->
<g id="icon_alarm" display="none">
  <circle r="15" fill="#ff0000"/>
  <text text-anchor="middle" y="5" fill="white">!</text>
</g>
```

- `icon_normal` → Visibility, Expression: `ins.getVariableValue("Temperature_C").value <= 70`
- `icon_alarm` → Visibility, Expression: `ins.getVariableValue("Temperature_C").value > 70`

---

## Blink (Blinking)

**Blink** causes an SVG element to blink when a condition is met. It provides a visual alert for situations that require attention (active alarm, critical value, communication error).

### Usage

| Field | Value |
|-------|-------|
| **Type** | Blink |
| **Suitable SVG Elements** | All |
| **Expression Type** | Tag (Boolean), Expression |

### How It Works

When `true` is returned, an SVG `<animate>` element is created and the element's opacity transitions between 0↔1. When `false` is returned, the animation is removed.

### SVG Preparation

```xml
<circle id="critical_alarm" cx="50" cy="50" r="20" fill="#ff0000"/>
```

### Expression Examples

**Boolean — Tag:**
```
AlarmActive
```

**Threshold — Expression:**
```javascript
var temp = ins.getVariableValue("Temperature_C").value;
return temp > 80; // Blink above 80°C
```

### Blink + Color Combination

Both Blink and Color can be applied to the same element:

1. **Color** element: color based on temperature (green → orange → red)
2. **Blink** element: blinking above 80°C

Result: Steady green in normal state, steady orange at 60°C, blinking red above 80°C.
