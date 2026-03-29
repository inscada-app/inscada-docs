---
title: "Set, Button & Click"
description: "Value writing, button, and click control animations"
sidebar:
  order: 17
---

The animation types on this page allow the operator to write values to variables from the SVG screen.

:::caution
Control types send commands to field devices. The user must have `SET_VARIABLE_VALUE` permission.
:::

## Set (Value Writing)

**Set** writes a predefined value to the target variable when the SVG element is clicked. It is used for simple controls such as motor start/stop, valve open/close.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Set |
| **Suitable SVG Elements** | All (any clickable element) |
| **Expression Type** | Set |

### SVG Preparation

```xml
<g cursor="pointer">
  <!-- START button -->
  <rect id="btn_start" x="10" y="10" width="100" height="40"
        rx="5" fill="#4CAF50"/>
  <text x="60" y="35" text-anchor="middle" fill="white"
        font-size="14" pointer-events="none">START</text>
</g>

<g cursor="pointer">
  <!-- STOP button -->
  <rect id="btn_stop" x="130" y="10" width="100" height="40"
        rx="5" fill="#f44336"/>
  <text x="180" y="35" text-anchor="middle" fill="white"
        font-size="14" pointer-events="none">STOP</text>
</g>
```

### Configuration

- `btn_start` → **Set**, target: `Motor_Run`, value: `{value: true}`
- `btn_stop` → **Set**, target: `Motor_Run`, value: `{value: false}`

When clicked, `ins.setVariableValue()` is called directly.

### Confirmation Dialog

A confirmation window can be shown before clicking by setting `confirm: true` in Props:

```
"Start the motor?" → Yes / No
```

---

## Button

**Button** operates as a Webix button component. Unlike Set: it has Webix button styling and can trigger more complex actions.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Button |
| **Suitable SVG Elements** | `<rect>` (foreignObject) |
| **Expression Type** | Button |

### Configuration (Props)

| Property | Description |
|----------|-------------|
| **label** | Button text |
| **css** | CSS class (color/style) |
| **action** | Click action |

---

## Click (Click Event)

**Click** runs custom JavaScript code when the SVG element is clicked. Unlike Set: instead of writing a fixed value, it can perform programmatic operations — confirmation dialog, calculation, writing to multiple variables.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Click |
| **Suitable SVG Elements** | All |
| **Expression Type** | Expression |

### Expression Examples

**Confirmed value writing:**
```javascript
if (confirm('Start the motor?')) {
    ins.setVariableValue('Motor_Run', {value: true});
    ins.writeLog("INFO", "Control", "Motor started by operator");
}
```

**Toggle (on/off):**
```javascript
var current = ins.getVariableValue("Pump_Running").value;
ins.setVariableValue("Pump_Running", {value: !current});
```

**Batch command:**
```javascript
if (confirm('Stop all pumps?')) {
    ins.setVariableValues({
        "Pump1_Run": {value: false},
        "Pump2_Run": {value: false},
        "Pump3_Run": {value: false}
    });
    ins.notify("warning", "Control", "All pumps stopped");
}
```

**Navigate to another animation:**
```javascript
// Open detail screen based on clicked motor
var motorId = 3;
// The Open animation type can also be used but Click
// allows more flexible parameter passing
```

---

## MouseDown / MouseUp (Press and Hold)

**MouseDown** and **MouseUp** trigger separate actions when the mouse button is pressed and released. Ideal for **jog** (momentary movement) controls.

### Usage

| Field | Value |
|-------|-------|
| **Type** | MouseDown / MouseUp |
| **Suitable SVG Elements** | All |
| **Expression Type** | Expression |

### Jog Control Example

```xml
<rect id="btn_jog_forward" width="80" height="40" fill="#2196F3" cursor="pointer"/>
<text x="40" y="25" text-anchor="middle" fill="white">JOG →</text>
```

- `btn_jog_forward` → **MouseDown**: `ins.setVariableValue("Jog_Forward", {value: true})`
- `btn_jog_forward` → **MouseUp**: `ins.setVariableValue("Jog_Forward", {value: false})`

The motor moves forward as long as you hold the button, and stops when you release it.

---

## MouseOver (Mouse Hover)

**MouseOver** triggers an action when the mouse hovers over an element.

### Usage

| Field | Value |
|-------|-------|
| **Type** | MouseOver |
| **Suitable SVG Elements** | All |
| **Expression Type** | Expression |

Unlike Tooltip: it can run JavaScript code, read variables, and update other elements.
