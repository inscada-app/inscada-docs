---
title: "Script API"
description: "Script scheduling, manual execution, status queries, and shared global objects"
sidebar:
  order: 7
---

Script API schedules / unschedules other scripts, runs them on demand, queries their status, and shares data between scripts. The shared data layer is Redis-backed and project-scoped.

## Script Metadata and Status

### `ins.getScript(name)` / `(projectName, name)`

Returns a script's definition — `RepeatableScriptResponseDto`.

```javascript
var s = ins.getScript("HourlyReport");
ins.consoleLog(s.getName() + " type=" + s.getType() + " period=" + s.getPeriod() + "ms");
```

Fields:

| Method | Type | Description |
| --- | --- | --- |
| `getName()` / `getDsc()` | `String` | Script name and description |
| `getCode()` | `String` | JavaScript source |
| `getType()` | `ScheduleType` | `PERIODIC` / `DAILY` / `ONCE` / `NONE` |
| `getPeriod()` | `Integer` | Period (ms) for periodic scripts |
| `getOffset()` | `Integer` | Offset (ms) for periodic scripts |
| `getDelay()` | `Integer` | Delay (ms) for once-type scripts |
| `getTime()` | `Date` | Time for daily scripts |
| `getLog()` | `Boolean` | Whether execution logging is on |
| `getProjectId()` | `String` | Project ID |

### `ins.getScriptStatus(name)` / `(projectName, name)`

Returns a `ScriptStatus` enum — two values:

| Value | Meaning |
| --- | --- |
| `"Scheduled"` | Attached to the scheduler |
| `"Not Scheduled"` | Not attached (can be run manually) |

```javascript
if (ins.getScriptStatus("HourlyReport") == "Not Scheduled") {
    ins.scheduleScript("HourlyReport");
}
```

## Script Control

### `ins.scheduleScript(name)` / `(projectName, name)`

Adds the script to the scheduler according to its `ScheduleType`. Has no effect on scripts with `type=NONE`.

```javascript
ins.scheduleScript("HourlyReport");
```

### `ins.cancelScript(name)` / `(projectName, name)`

**Removes the script from the scheduler** — future triggers are cancelled, but an execution that is currently running is **not** interrupted (it runs to completion).

```javascript
ins.cancelScript("HourlyReport");
```

### `ins.executeScript(name)` / `(projectName, name)`

Runs the script **immediately** and returns its return value. Synchronous — blocks until the script finishes.

```javascript
var dailyTotal = ins.executeScript("Calculate_DailyTotal");
ins.setVariableValue("DailyProduction_kWh", { value: dailyTotal });
```

:::note
The caller and callee share the same statement / timeout limits; the callee's execution time is charged against the caller's total budget. A missing script throws `ScriptException`.
:::

## Global Object — Inter-Script Sharing

Scripts are isolated from each other (no shared memory). To bridge this, `setGlobalObject` / `getGlobalObject` provide a Redis-backed key-value store. Keys are project-scoped (`project:<projectId>:global-object:<name>`).

### `ins.setGlobalObject(name, object)`

Stores the object **indefinitely** (no TTL).

```javascript
ins.setGlobalObject("daily_counter", 42);
ins.setGlobalObject("shift_data", {
    shift: "A",
    count: 150,
    startTime: ins.now().toString()
});
```

### `ins.setGlobalObject(name, object, ms)`

Stores the object **with a TTL** — auto-deleted after `ms` milliseconds (Redis `SET PX`).

```javascript
// Cache that vanishes after 60 seconds
ins.setGlobalObject("temp_cache", { value: 99 }, 60000);
```

### `ins.getGlobalObject(name)`

Returns the object, or `null` if missing. Does not touch the TTL.

```javascript
var counter = ins.getGlobalObject("daily_counter");
// → 42 or null
```

### `ins.getGlobalObject(name, ms)`

Returns the object **and resets the TTL to `ms`** (sliding TTL — Redis `PEXPIRE`). Typical use: "Each read keeps the object alive."

```javascript
// Session-like data: each read extends liveness by 5 minutes
var session = ins.getGlobalObject("user_session", 5 * 60 * 1000);
```

:::note
Objects are serialized to JSON. Dates become strings, functions and class instances are lost. Only plain objects, arrays, numbers, strings and booleans are safe.
:::

:::note
Redis sharing makes global objects visible from **every node** in a cluster; behaviour across restarts depends on the Redis persistence settings.
:::

## Common Patterns

### Backend → UI Data Hand-off

```javascript
// Backend script (every 10 s):
function main() {
    ins.setGlobalObject("dashboard_summary", {
        power:   ins.getVariableValue("ActivePower_kW").value,
        voltage: ins.getVariableValue("Voltage_V").value,
        updatedAt: ins.now().getTime()
    }, 30000);   // 30s TTL — don't let it go stale
}
main();
```

A custom HTML widget on the UI side can read the same key via client-side `Inscada.*`.

### Rate Limiting

```javascript
function main() {
    var last = ins.getGlobalObject("report_last_run");
    var now  = ins.now().getTime();

    if (last && (now - last) < 3600000) {
        ins.consoleLog("Less than 1 hour, skipping");
        return;
    }

    ins.executeScript("generate_hourly_report");
    ins.setGlobalObject("report_last_run", now);
}
main();
```

### Flag / Mutex

```javascript
function main() {
    if (ins.getGlobalObject("maintenance_mode")) {
        ins.consoleLog("Maintenance mode on — script skipped");
        return;
    }
    // normal work...
}
main();
```
