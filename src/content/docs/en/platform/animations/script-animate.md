---
title: "Script & Animate"
description: "Custom JavaScript, CSS animation, access control, 3D, QR code"
sidebar:
  order: 20
---

## Script

**Script** runs custom JavaScript code associated with an SVG element. It runs periodically every animation cycle (duration). It is used for complex logic that cannot be achieved with other animation types.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Script |
| **Suitable SVG Elements** | All |
| **Expression Type** | Expression |

### Expression Examples

**Updating multiple DOM elements:**
```javascript
var power = ins.getVariableValue("ActivePower_kW").value;
var voltage = ins.getVariableValue("Voltage_V").value;
var current = voltage > 0 ? (power * 1000 / voltage) : 0;

// Display the calculated current
var el = document.getElementById("calc_current");
el.textContent = current.toFixed(1) + " A";

// Set its color
el.setAttribute("fill", current > 50 ? "#ff0000" : "#00cc00");
```

**Dynamic SVG creation:**
```javascript
var values = ins.getVariableValues(["Temp1", "Temp2", "Temp3", "Temp4"]);
var container = document.getElementById("temp_bars");
container.innerHTML = "";

var names = ["Temp1", "Temp2", "Temp3", "Temp4"];
for (var i = 0; i < names.length; i++) {
    var val = values[names[i]].value;
    var height = val * 2; // 0-100 → 0-200px
    var color = val > 70 ? "#ff0000" : val > 50 ? "#ff8800" : "#00cc00";

    var rect = document.createElementNS("http://www.w3.org/2000/svg", "rect");
    rect.setAttribute("x", i * 50 + 10);
    rect.setAttribute("y", 200 - height);
    rect.setAttribute("width", 35);
    rect.setAttribute("height", height);
    rect.setAttribute("fill", color);
    container.appendChild(rect);
}
```

:::tip
For simple bindings, prefer ready-made types like Get, Color, Bar. Script runs every cycle and DOM manipulation affects performance.
:::

### FormScript

**FormScript** is a form-based script component. It performs batch operations by collecting multiple inputs from the user.

| Usage | Example |
|-------|---------|
| Recipe change | Update multiple setpoints in a single form |
| Report parameters | Generate report with date range + variable selection |
| Batch parameter setting | Edit PID parameters (P, I, D) in a single form |

---

## Animate (CSS/SVG Animation)

**Animate** starts/stops a CSS animation or SVG SMIL animation based on a condition. It is used for visual effects such as rotation, vibration, pulse, and blinking.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Animate |
| **Suitable SVG Elements** | `<g>`, `<path>`, `<circle>` |
| **Expression Type** | Tag (Boolean), Expression |

### SVG + CSS Example — Spinning Fan

```xml
<style>
  @keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }
  .spinning { animation: spin 1s linear infinite; transform-origin: center; }
</style>

<g id="fan_blades" transform="translate(100,100)">
  <line x1="-30" y1="0" x2="30" y2="0" stroke="#333" stroke-width="4"/>
  <line x1="0" y1="-30" x2="0" y2="30" stroke="#333" stroke-width="4"/>
  <line x1="-21" y1="-21" x2="21" y2="21" stroke="#333" stroke-width="4"/>
  <line x1="21" y1="-21" x2="-21" y2="21" stroke="#333" stroke-width="4"/>
</g>
```

Expression: `Fan_Running` (Tag, Boolean)
- `true` → `.spinning` class is added to `fan_blades` → fan spins
- `false` → class is removed → fan stops

---

## Access (Permission Control)

**Access** shows or hides an SVG element based on user permissions. It is used to show security-sensitive control buttons only to authorized users.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Access |
| **Suitable SVG Elements** | `<g>` (group control buttons) |
| **Expression** | Required permission name |

### Example

```xml
<!-- Only visible to users with SET_VARIABLE_VALUE permission -->
<g id="control_buttons">
  <rect id="btn_start" ... />
  <rect id="btn_stop" ... />
</g>
```

- `control_buttons` → **Access**, Expression: `SET_VARIABLE_VALUE`
- Users without the permission cannot see this group at all

---

## Three (3D Visualization)

**Three** provides Three.js-based 3D model embedding.

| Field | Value |
|-------|-------|
| **Type** | Three |
| **Suitable SVG Elements** | `<rect>` (foreignObject) |

Usage: 3D facility model, equipment visualization, virtual tour.

---

## QRCodeGeneration (QR Code Generation)

**QRCodeGeneration** generates a QR code from a variable value or text.

| Field | Value |
|-------|-------|
| **Type** | QRCodeGeneration |
| **Expression Type** | Expression, Text |

```javascript
// Convert device information to QR code
var sn = ins.getVariableValue("Serial_Number").value;
return "https://inscada.com/device/" + sn;
```

Usage: Device identification code, maintenance form URL, inventory label.

## QRCodeScan (QR Code Scanning)

**QRCodeScan** opens a camera-based QR code scanner component. It passes the scanned data to a variable or script.

Usage: Device recognition from mobile device, maintenance record entry, field personnel identity verification.
