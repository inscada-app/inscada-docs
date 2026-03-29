---
title: "Get"
description: "Displaying variable values as text"
sidebar:
  order: 10
---

**Get** is the most basic animation type that updates the content of an SVG text element with a variable value. It is used for all text-based displays such as numeric indicators, labels, and status text.

## Usage

| Field | Value |
|-------|-------|
| **Type** | Get |
| **Suitable SVG Elements** | `<text>`, `<tspan>` |
| **Expression Type** | Tag, Expression, Switch, Text |

## SVG Preparation

```xml
<text id="power_display" x="100" y="50"
      font-size="24" fill="#333" text-anchor="middle">--</text>
```

## Configuration Examples

### Simple — Direct Binding with Tag

Expression Type: **Tag**
```
ActivePower_kW
```
Result: `359.91` (raw value of the variable)

### Formatted — With Expression

Expression Type: **Expression**
```javascript
var val = ins.getVariableValue("ActivePower_kW");
return val.value.toFixed(1) + " kW";
```
Result: `359.9 kW`

### Unit and Decimal

```javascript
var val = ins.getVariableValue("Temperature_C");
return val.value.toFixed(1) + " °C";
```
Result: `45.2 °C`

### Conditional Text — With Switch

Expression Type: **Switch**
```
0 → Stopped
1 → Running
2 → Fault
3 → Maintenance
```

### Boolean Status Text

```javascript
var status = ins.getVariableValue("GridStatus").value;
return status ? "ONLINE" : "OFFLINE";
```

### Timestamp

```javascript
var val = ins.getVariableValue("ActivePower_kW");
var d = new Date(val.dateInMs);
var h = ("0" + d.getHours()).slice(-2);
var m = ("0" + d.getMinutes()).slice(-2);
var s = ("0" + d.getSeconds()).slice(-2);
return h + ":" + m + ":" + s;
```
Result: `14:32:05`

### Multiple Variables

```javascript
var p = ins.getVariableValue("ActivePower_kW").value;
var v = ins.getVariableValue("Voltage_V").value;
var i = ins.getVariableValue("Current_A").value;
return p.toFixed(0) + " kW | " + v.toFixed(0) + " V | " + i.toFixed(1) + " A";
```
Result: `360 kW | 235 V | 36.2 A`

## GetSymbol

The **GetSymbol** type loads an SVG symbol from the Space-level Symbol library. It is the visual symbol version of Get instead of text.

| Field | Value |
|-------|-------|
| **Type** | GetSymbol |
| **Expression Type** | Expression |
| **Expression** | Symbol name |

Usage: Displaying dynamic icons based on device type.
