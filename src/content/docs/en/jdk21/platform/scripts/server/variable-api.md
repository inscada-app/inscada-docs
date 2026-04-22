---
title: "Variable API"
description: "Live and historical variable values, bulk read/write, statistics and variable metadata"
sidebar:
  order: 1
---

Variable API reads **live** and **historical** values of inSCADA variables from a script, supports bulk read/write, statistics queries and variable metadata management.

## Reading Live Values

### `ins.getVariableValue(name)` / `ins.getVariableValue(projectName, name)`

Returns the current value of a variable as a `VariableValueDto`. Data is served from an in-memory cache (< 1 ms).

```javascript
var v = ins.getVariableValue("ActivePower_kW");
var power = v.value;          // or v.getValue()
var ts = v.dateInMs;          // epoch ms
```

Read from another project:

```javascript
var v = ins.getVariableValue("other_project", "pressure");
```

### `ins.getVariableValue(name, index)` / `ins.getVariableValue(projectName, name, index)`

Returns the element at the given index of an array variable.

```javascript
var el3 = ins.getVariableValue("measurements_array", 3);   // 4th element
```

### `ins.getVariableValues(name, fromIndex, toIndex)` / `(projectName, name, fromIndex, toIndex)`

Returns a range of array-variable elements (inclusive) — `Collection<VariableValueDto>`.

```javascript
var range = ins.getVariableValues("measurements_array", 0, 9);   // first 10 elements
```

### `ins.getVariableValues(names[])` / `(projectName, names[])`

Reads multiple variables in a single call — returns `Map<String, VariableValueDto>`. Much faster than reading them one by one.

```javascript
var vals = ins.getVariableValues(["ActivePower_kW", "Voltage_V", "Current_A"]);
var p = vals.ActivePower_kW.value;
var u = vals.Voltage_V.value;
var i = vals.Current_A.value;
```

### `ins.getProjectVariableValues()` / `(projectName)`

Returns live values for **every** variable in the project — `Map<String, VariableValueDto>`.

```javascript
var all = ins.getProjectVariableValues();
Object.keys(all).forEach(function(name) {
    ins.consoleLog(name + " = " + all[name].value);
});
```

### `VariableValueDto` Fields

| Field / Method | Type | Description |
| --- | --- | --- |
| `getValue()` / `.value` | `Object` | Scaled (engineering) value |
| `getDate()` / `.date` | `Date` | Value timestamp |
| `getDateInMs()` / `.dateInMs` | `Long` | Same timestamp — epoch ms |
| `getDttm()` / `.dttm` | `Date` | Server reception time |
| `getTime()` / `.time` | `Long` | Server reception time — epoch ms |
| `getVariableShortInfo()` / `.variableShortInfo` | `VariableShortInfoDto` | Variable metadata: `name`, `dsc`, `connection`, `device`, `frame`, `project` |

:::note
The `flags` (e.g. `scaled`) and `extras` (e.g. `raw_value`) fields are not exposed to scripts (no `@HostAccess.Export`) — `v.flags` / `v.extras` are not directly accessible; they only appear in JSON serialization (`ins.toJSONStr(v)` or in REST responses).
:::

## Writing Values

### `ins.setVariableValue(name, details)` / `(projectName, name, details)`

Writes to a variable. `details` is a `Map<String, Object>` that must contain at least a `value` key.

```javascript
ins.setVariableValue("Temperature_C", { value: 55.0 });
ins.setVariableValue("GridStatus", { value: true });
ins.setVariableValue("other_project", "target_temp", { value: 80.0 });
```

### `ins.setVariableValues(map)` / `(projectName, map)`

Bulk write — `Map<String, Map<String, Object>>`.

```javascript
ins.setVariableValues({
    "Temperature_C": { value: 42.5 },
    "Voltage_V": { value: 228.0 },
    "PumpRun": { value: true }
});
```

### `ins.mapVariableValue(src, dest)` / `(src, dest, defaultValue)` / projectName variants

Copies the live value of `src` into `dest`. If `src` is `null` / unread, the optional `defaultValue` is used.

```javascript
ins.mapVariableValue("Temperature_C", "display_temp");
ins.mapVariableValue("Temperature_C", "display_temp", 0);
```

### `ins.toggleVariableValue(name)` / `(projectName, name)`

Flips a boolean variable (`true` ↔ `false`).

```javascript
ins.toggleVariableValue("GridStatus");
```

## Historical Data (Logged Values)

### `ins.getLoggedVariableValuesByPage(names[], startDate, endDate, page, pageSize)`

Returns paged log records for the given interval — `Collection<LoggedVariableValueDto>`. Results are sorted **newest-to-oldest** (DESC).

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 300000);   // last 5 minutes

var logs = ins.getLoggedVariableValuesByPage(
    ["ActivePower_kW"],
    start, end,
    0,    // page number
    100   // page size
);

logs.forEach(function(r) {
    ins.consoleLog(r.getDttm() + " → " + r.getValue());
});
```

### `ins.getLoggedVariableValuesByPageAsc(names[], startDate, endDate, page, pageSize)`

Same as above but sorted **oldest-to-newest** (ASC).

### `LoggedVariableValueDto` Fields

| Field / Method | Type | Description |
| --- | --- | --- |
| `getName()` / `.name` | `String` | Variable name |
| `getValue()` / `.value` | `Double` | Numeric value |
| `getTextValue()` / `.textValue` | `String` | Text value (if the variable is a string) |
| `getDttm()` / `.dttm` | `Date` | Record timestamp |
| `getTime()` / `.time` | `Long` | Epoch ms |
| `getVariableId()` / `.variableId` | `String` | Variable ID |
| `getProject()` / `.project` | `String` | Project name |
| `getProjectId()` / `.projectId` | `String` | Project ID |

### `ins.getLoggedVariableNames()` / `(projectName)`

List of variables that are being logged — useful to discover what's recorded.

```javascript
var loggedOnes = ins.getLoggedVariableNames();
ins.consoleLog(loggedOnes.size() + " variables are being logged");
```

## Historical Statistics

All share the same shape: `(variableNames[], startDate, endDate)` with an optional `projectName` prefix.

### `ins.getLoggedVariableValueStats(names[], startDate, endDate)`

One stats set across the whole interval — `Map<String, LoggedVariableValueStatsDto>`.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 3600000);   // last hour

var stats = ins.getLoggedVariableValueStats(["ActivePower_kW"], start, end);
var s = stats.ActivePower_kW;
ins.consoleLog("avg=" + s.getAvgValue() + " min=" + s.getMinValue() + " max=" + s.getMaxValue());
```

### `ins.getLoggedHourlyVariableValueStats(names[], startDate, endDate)`

**Hourly** groups — `Map<String, List<LoggedVariableValueStatsDto>>`. One list per variable with an entry per hour.

```javascript
var hourly = ins.getLoggedHourlyVariableValueStats(["ActivePower_kW"], start, end);
hourly.ActivePower_kW.forEach(function(s) {
    ins.consoleLog(s.getDttm() + " avg=" + s.getAvgValue());
});
```

### `ins.getLoggedDailyVariableValueStats(names[], startDate, endDate)`

**Daily** groups — same shape.

### `ins.getLoggedVariableValueStatsByInterval(names[], startDate, endDate, interval)`

Arbitrary bucket size — `interval` in milliseconds. Returns `Collection<LoggedVariableValueStatsDto>`.

```javascript
// 5-minute buckets
var bucket5min = ins.getLoggedVariableValueStatsByInterval(
    ["ActivePower_kW"], start, end, 5 * 60 * 1000
);
```

### `LoggedVariableValueStatsDto` Fields

| Field / Method | Type | Description |
| --- | --- | --- |
| `getName()` / `.name` | `String` | Variable name |
| `getVariableId()` / `.variableId` | `String` | Variable ID |
| `getDttm()` / `.dttm` | `Date` | Bucket start time |
| `getMinValue()` | `Double` | Minimum |
| `getMaxValue()` | `Double` | Maximum |
| `getAvgValue()` | `Double` | Average |
| `getSumValue()` | `Double` | Sum |
| `getCountValue()` | `Double` | Record count |
| `getMedianValue()` | `Double` | Median |
| `getMiddleValue()` | `Double` | Midpoint (min+max)/2 |
| `getFirstValue()` | `Double` | First value |
| `getLastValue()` | `Double` | Last value |
| `getMaxDiffValue()` | `Double` | Max − Min |
| `getLastFirstDiffValue()` | `Double` | Last − First |

## Variable Metadata

### `ins.getVariables()` / `(projectName)`

Every variable definition in the project — `Collection<VariableResponseDto>`.

```javascript
var list = ins.getVariables();
ins.consoleLog("Total " + list.size() + " variables");
```

### `ins.getVariablesByConnectionName(connectionName)`

Variables that belong to a specific connection.

### `ins.getVariablesByDeviceName(connectionName, deviceName)`

Variables that belong to a specific device.

### `ins.getVariablesByFrameName(connectionName, deviceName, frameName)`

Variables that belong to a specific frame.

```javascript
var pLoc = ins.getVariablesByFrameName("MODBUS-PLC", "Device1", "HoldingRegs_0_100");
pLoc.forEach(function(v) {
    ins.consoleLog(v.getName() + " — " + v.getDsc());
});
```

### `ins.getVariable(name)`

Metadata for one variable — `VariableResponseDto`.

```javascript
var v = ins.getVariable("ActivePower_kW");
ins.consoleLog(v.getName() + " — " + v.getDsc());
```

### `ins.updateVariable(name, dto)`

Updates a variable's configuration — requires a **full DTO**. Typical pattern: read with `getVariable`, mutate a field, write back with `updateVariable`.

```javascript
var v = ins.getVariable("ActivePower_kW");
v.setDsc("Total active power — updated");
// Other setters: v.setActiveFlag(true), v.setLogType(...), etc.
ins.updateVariable("ActivePower_kW", v);
```

## Example: Write the Last-hour Average Power

```javascript
function main() {
    var end = ins.now();
    var start = ins.getDate(end.getTime() - 3600000);

    var stats = ins.getLoggedVariableValueStats(["ActivePower_kW"], start, end);
    var avg = stats.ActivePower_kW.getAvgValue();

    ins.setVariableValue("ActivePower_1h_Avg", { value: avg });
    ins.writeLog("INFO", "PowerStats", "1h avg = " + avg.toFixed(2) + " kW");
}
main();
```
