---
title: "Slider & Input"
description: "Slider and text/number input controls"
sidebar:
  order: 18
---

## Slider

**Slider** is used to set analog values with a draggable slider. For continuous value controls such as setpoint, speed adjustment, temperature target, and dimmer.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Slider |
| **Suitable SVG Elements** | `<rect>` (foreignObject) |
| **Expression Type** | Tag, Expression |

### Configuration (Props)

| Property | Description | Example |
|----------|-------------|---------|
| **min** | Minimum value | 0 |
| **max** | Maximum value | 100 |
| **step** | Step size | 1 |
| **title** | Title | "Temperature Setpoint" |

### SVG Preparation

```xml
<foreignObject id="temp_slider" x="50" y="200" width="300" height="60"/>
```

### Configuration

- Expression Type: **Tag** → `Temperature_Setpoint`
- Props: min=0, max=100, step=0.5

When the operator drags the slider, the value is written to the variable immediately.

### Advanced with Expression

```javascript
// Logging when value is written
var newValue = value; // value from the slider
ins.setVariableValue("Temperature_Setpoint", {value: newValue});
ins.writeLog("INFO", "Setpoint", "Temperature setpoint: " + newValue + "°C");
return newValue;
```

---

## Input (Input Field)

**Input** creates a text or number input field. The operator enters a value from the keyboard and confirms with Enter.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Input |
| **Suitable SVG Elements** | `<rect>` (foreignObject) |
| **Expression Type** | Tag, Expression |

### Configuration (Props)

| Property | Description | Example |
|----------|-------------|---------|
| **type** | Input type | `number`, `text` |
| **min** | Minimum value (number) | 0 |
| **max** | Maximum value (number) | 100 |
| **placeholder** | Text shown when empty | "Enter setpoint..." |

### SVG Preparation

```xml
<foreignObject id="setpoint_input" x="100" y="150" width="150" height="35"/>
```

### Usage Scenarios

**Numeric setpoint:**
- Expression Type: Tag → `Temperature_Setpoint`
- Props: type=number, min=0, max=100

**Recipe name:**
- Expression Type: Tag → `Recipe_Name`
- Props: type=text, placeholder="Recipe name..."

**Parameter input:**
- Expression Type: Expression
```javascript
var val = parseFloat(value);
if (val < 0 || val > 100) {
    ins.notify("error", "Error", "Value must be between 0-100!");
    return;
}
ins.setVariableValue("Setpoint", {value: val});
ins.notify("success", "OK", "Setpoint updated: " + val);
```
