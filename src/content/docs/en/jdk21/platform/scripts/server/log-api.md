---
title: "Log API"
description: "Write and query the project audit log"
sidebar:
  order: 10
---

Log API lets a script append severity-tagged (info / warn / error) records to the project audit log and query records filtered by time, severity, and activity. Entries written here appear on the platform's Log screen and through REST endpoints.

## `ins.writeLog(type, activity, msg)` / `(projectName, type, activity, msg)`

Appends a new record to the project audit log. `projectName` defaults to the current project.

| Parameter | Type | Description |
| --- | --- | --- |
| `type` | `String` | Log severity — see below |
| `activity` | `String` | Activity label (free-form) |
| `msg` | `String` | Log message |

### Valid `type` Values

`type` must be one of three **lowercase** values:

| `type` | Severity |
| --- | --- |
| `"error"` | Error |
| `"warn"` | Warning |
| any other value (e.g. `"info"`, empty, `null`) | Information (default) |

```javascript
ins.writeLog("info", "Setpoint Change", "Temperature_C: 48.0 → 55.0");
ins.writeLog("warn", "Network", "Modbus timeout — retry scheduled");
ins.writeLog("error", "BatchProcess", "Checksum mismatch at record 42");
```

:::caution
Uppercase `"INFO"` / `"WARNING"` / `"ERROR"` do not match and fall through to the default (INFO). `"warn"` (four characters) is correct — `"warning"` is not.
:::

## `ins.getLogsByPage(...)`

Returns paged log records — `Collection<LogEntryDto>`. Three overloads are available.

### `ins.getLogsByPage(startDate, endDate, page, pageSize)`

Simplest — current project, no filters.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 3600000);   // last hour

var logs = ins.getLogsByPage(start, end, 0, 50);
```

### `ins.getLogsByPage(activity, logSeverity, startDate, endDate, page, pageSize)`

Adds activity and severity filters.

```javascript
var errors = ins.getLogsByPage("BatchProcess", "Error", start, end, 0, 100);
errors.forEach(function(e) {
    ins.consoleLog(e.getDttm() + " [" + e.getLogSeverity() + "] " + e.getMsg());
});
```

### `ins.getLogsByPage(projectName, activity, logSeverity, startDate, endDate, page, pageSize)`

Targets a specific project.

```javascript
var logs = ins.getLogsByPage("other_project", null, null, start, end, 0, 20);
```

### Filter Parameters

| Parameter | Description |
| --- | --- |
| `activity` | Exact match; pass `null` to include all activities |
| `logSeverity` | `"Information"`, `"Warning"`, `"Error"` — the **full** value (different from writeLog); pass `null` to include all |

### `LogEntryDto` Fields

| Method | Type | Description |
| --- | --- | --- |
| `getActivity()` | `String` | Activity label |
| `getMsg()` | `String` | Log message |
| `getLogSeverity()` | `LogSeverity` | `"Information"` / `"Warning"` / `"Error"` |
| `getDttm()` | `Date` | Log timestamp |
| `getTime()` | `Long` | Epoch ms |
| `getProject()` / `getProjectId()` | `String` | Project name / ID |

## Example: Report Last-24h Error Count

```javascript
function main() {
    var end = ins.now();
    var start = ins.getDate(end.getTime() - 86400000);

    var errors = ins.getLogsByPage(null, "Error", start, end, 0, 1000);
    ins.setVariableValue("ErrorCount_24h", { value: errors.size() });

    if (errors.size() > 50) {
        ins.notify("error", "Log alarm", "Last 24h: " + errors.size() + " errors");
    }
}
main();
```
