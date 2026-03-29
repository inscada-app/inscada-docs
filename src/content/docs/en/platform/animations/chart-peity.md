---
title: "Chart & Peity"
description: "Chart components and inline sparkline mini charts"
sidebar:
  order: 16
---

## Chart

**Chart** places an interactive chart component inside the SVG screen. It visualizes historical data or real-time value series.

![Active & Reactive Power Chart](../../../../../assets/docs/editor-preview.png)

### Usage

| Field | Value |
|-------|-------|
| **Type** | Chart |
| **Suitable SVG Elements** | `<rect>` (converted to foreignObject) |
| **Expression Type** | Collection, Expression |

### Chart Types

| Type | Usage |
|------|-------|
| **line** | Time series, trend tracking |
| **bar** | Comparison, periodic summary |
| **area** | Cumulative values, area chart |
| **pie** | Distribution display |

### Configuration (Props)

| Property | Description |
|----------|-------------|
| **type** | Chart type (line, bar, area, pie) |
| **colors** | Series colors |
| **xAxis** | X axis settings |
| **yAxis** | Y axis settings |
| **legend** | Show/hide legend |

### Expression Example — Historical Data Chart

```javascript
var endMs = ins.now().getTime();
var startMs = endMs - (60 * 60 * 1000); // Last 1 hour

var logs = ins.getLoggedVariableValuesByPage(
    ['ActivePower_kW'],
    ins.getDate(startMs), ins.getDate(endMs), 0, 30
);
var logs2 = ins.getLoggedVariableValuesByPage(
    ['ReactivePower_kVAR'],
    ins.getDate(startMs), ins.getDate(endMs), 0, 30
);

var labels = [];
var active = [];
var reactive = [];

for (var i = logs.length - 1; i >= 0; i--) {
    labels.push(logs[i].dttm);
    active.push(logs[i].value);
}
for (var j = logs2.length - 1; j >= 0; j--) {
    reactive.push(logs2[j].value);
}

return {
    type: 'area',
    labels: labels,
    dataset: {
        0: {
            name: 'Active Power (kW)',
            data: active,
            color: '#00d4ff',
            fill: true
        },
        1: {
            name: 'Reactive Power (kVAR)',
            data: reactive,
            color: '#818cf8',
            fill: true
        }
    },
    xAxes: { 0: { labels: labels } },
    options: {}
};
```

This example produces the chart shown in the Preview Animation screenshot above.

### Expression Example — Real-Time Distribution (Pie)

```javascript
var power = ins.getVariableValue("ActivePower_kW").value;
var reactive = ins.getVariableValue("ReactivePower_kVAR").value;

return {
    type: 'pie',
    dataset: {
        0: { name: 'Active', data: [power], color: '#3498db' },
        1: { name: 'Reactive', data: [reactive], color: '#e74c3c' }
    }
};
```

---

## Peity (Sparkline Mini Chart)

**Peity** creates small inline sparkline charts. A compact trend display next to text — visualizes the direction of a value without taking up much space.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Peity |
| **Suitable SVG Elements** | `<rect>` (foreignObject) |
| **Expression Type** | Collection |

### Peity Types

| Type | Appearance |
|------|------------|
| **line** | Line sparkline |
| **bar** | Mini bar chart |
| **pie** | Mini pie chart |
| **donut** | Mini donut chart |

### Usage Scenario

Adding a mini trend chart of the last 10 values next to a variable value:

```xml
<g transform="translate(200, 30)">
  <text id="power_value" font-size="20">350 kW</text>
  <foreignObject id="power_sparkline" x="100" y="-10" width="80" height="25"/>
</g>
```

- `power_value` → Get, Tag: `ActivePower_kW`
- `power_sparkline` → Peity, line sparkline with last 10 values

---

## Datatable (Table)

**Datatable** places an interactive table inside the SVG screen. It operates as a Webix datatable component.

### Usage

| Field | Value |
|-------|-------|
| **Type** | Datatable |
| **Suitable SVG Elements** | `<rect>` (foreignObject) |
| **Expression Type** | Collection, Expression |

### Configuration (Props)

| Property | Description |
|----------|-------------|
| **columns** | Column definitions (header, width, format) |
| **autoheight** | Auto height |
| **select** | Row selection |

### Expression Example — Variable Monitoring Table

```javascript
var names = ["ActivePower_kW", "Voltage_V", "Current_A", "Frequency_Hz"];
var rows = [];
for (var i = 0; i < names.length; i++) {
    var val = ins.getVariableValue(names[i]);
    rows.push({
        name: names[i],
        value: val.value.toFixed(2),
        unit: val.variableShortInfo.dsc,
        time: new Date(val.dateInMs).toLocaleTimeString()
    });
}
return rows;
```
