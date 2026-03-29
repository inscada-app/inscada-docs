---
title: "Open, Iframe & Faceplate"
description: "Screen navigation, external URL embedding, and faceplate placement"
sidebar:
  order: 19
---

## Open (Screen Navigation)

**Open** navigates to another animation screen when clicked. It is used to build hierarchical navigation between SCADA screens.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Open |
| **Suitable SVG Elements** | All (clickable) |
| **Expression Type** | Animation, Animation Popup |

### SVG Preparation

```xml
<g id="goto_motor_detail" cursor="pointer">
  <rect width="120" height="35" rx="5" fill="#1CA1C1"/>
  <text x="60" y="22" text-anchor="middle" fill="white" font-size="13">
    Motor Detail →
  </text>
</g>
```

### Expression Types

| Expression Type | Behavior |
|----------------|----------|
| **Animation** | Closes the current screen and opens the target animation |
| **Animation Popup** | Opens the target animation as a modal popup (main screen stays in the background) |

### Navigation Hierarchy Example

```
Main Screen (Overview)
├── [Open] → Motor Section
│   ├── [Open] → Motor 1 Detail
│   └── [Open] → Motor 2 Detail
├── [Open] → Pump Section
└── [Open Popup] → Alarm Summary (modal)
```

### Parameter Passing

Placeholder parameters can be passed to the animation opened via Open:

```
Target: Motor_Detail
Parameters: motor_id=1, motor_name=Motor 1
```

The `{motor_id}` and `{motor_name}` placeholders in the target screen are filled with these values.

---

## Iframe (External URL Embedding)

**Iframe** embeds an external web page inside the SVG screen. It integrates content such as Grafana dashboards, IP cameras, external web applications, and PDF documents into the SCADA screen.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Iframe |
| **Suitable SVG Elements** | `<rect>` (foreignObject), `<g>` |
| **Expression Type** | Url |

### SVG Preparation

```xml
<rect id="grafana_embed" x="10" y="300" width="600" height="400"
      fill="#f5f5f5" stroke="#ddd"/>
```

### Expression Examples

**Static URL:**
```
https://grafana.company.com/d/energy/panel?orgId=1&kiosk
```

**IP Camera:**
```
http://192.168.1.50/video.cgi
```

**Dynamic URL — Expression:**
```javascript
var projectId = 153;
return "https://grafana.company.com/d/energy?var-project=" + projectId + "&kiosk";
```

### Usage Scenarios

| Content | URL Format |
|---------|------------|
| Grafana Dashboard | `https://grafana/d/xxx?kiosk` |
| IP Camera MJPEG | `http://camera-ip/video` |
| PDF Report | `/files/reports/daily.pdf` |
| Custom Menu HTML | Platform internal Custom Menu page |
| External Web App | Any web application |

---

## Faceplate (Component Placement)

**Faceplate** places a pre-designed reusable SVG component into the animation screen. It allows the same symbol to be used multiple times with different parameters.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Faceplate |
| **Suitable SVG Elements** | `<g>`, `<rect>`, `<image>` |
| **Expression Type** | Faceplate |

### SVG Preparation

```xml
<!-- Motor 1 area -->
<g id="motor1_area" transform="translate(100, 200)"/>

<!-- Motor 2 area -->
<g id="motor2_area" transform="translate(350, 200)"/>

<!-- Motor 3 area -->
<g id="motor3_area" transform="translate(600, 200)"/>
```

### Configuration

The same Faceplate for each area, with different placeholder values:

**motor1_area:**
- Faceplate: `Motor_Standard`
- Placeholders: `{motor_name}=Motor 1, {speed_var}=M1_Speed, {status_var}=M1_Status`

**motor2_area:**
- Faceplate: `Motor_Standard`
- Placeholders: `{motor_name}=Motor 2, {speed_var}=M2_Speed, {status_var}=M2_Status`

**motor3_area:**
- Faceplate: `Motor_Standard`
- Placeholders: `{motor_name}=Motor 3, {speed_var}=M3_Speed, {status_var}=M3_Status`

3 motors, 1 faceplate design, 3 placeholder sets.

### Menu (Open Menu)

The **Menu** type opens a Custom Menu page when clicked.

| Field | Value |
|-------|-------|
| **Type** | Menu |
| **Expression Type** | Custom Menu |
| **Expression** | Custom Menu name |

Detailed information: [Faceplate →](/docs/tr/platform/faceplates/) | [Custom Menus →](/docs/tr/platform/custom-menus/)
