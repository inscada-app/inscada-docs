---
title: "Alarm API"
description: "Query alarms and alarm groups, activate / deactivate, fired alarm history, acknowledge / comment / force-off actions"
sidebar:
  order: 3
---

Alarm API queries alarms and alarm groups, activates or deactivates groups, fetches fired-alarm history, and runs acknowledge / comment / force-off actions on individual alarms.

## Alarm and Group Metadata

### `ins.getAlarm(name)` / `(projectName, name)`

Returns a single alarm's definition — `AlarmResponseDto`.

```javascript
var a = ins.getAlarm("HighTemperature");
ins.consoleLog(a.getName() + " (" + a.getType() + ") — delay=" + a.getDelay());
```

### `ins.getAlarmGroup(name)` / `(projectName, name)`

Returns a single alarm group's definition — `AlarmGroupResponseDto`.

```javascript
var g = ins.getAlarmGroup("Temperature_Alarms");
ins.consoleLog(g.getName() + " priority=" + g.getPriority() + " scan=" + g.getScanTimeInMillis() + "ms");
```

### `AlarmResponseDto` Fields

| Method | Type | Description |
| --- | --- | --- |
| `getName()` | `String` | Alarm name |
| `getDsc()` | `String` | Description |
| `getGroupId()` | `String` | Group ID |
| `getType()` | `String` | Type (`DIGITAL`, `ANALOG_HIGH_HIGH`, `ANALOG_HIGH`, `ANALOG_LOW`, `ANALOG_LOW_LOW`, `ANALOG_SET_POINT`, `CUSTOM`) |
| `getDelay()` | `Integer` | Trigger delay (seconds) |
| `getIsActive()` | `Boolean` | Whether the alarm definition is active |
| `getPart()` | `String` | Partition key inside the group |
| `getOnTimeVariableId()` / `getOffTimeVariableId()` | `String` | Variable IDs where on/off timestamps get written |

### `AlarmGroupResponseDto` Fields

| Method | Type | Description |
| --- | --- | --- |
| `getName()` | `String` | Group name |
| `getDsc()` | `String` | Description |
| `getPriority()` | `Short` | Priority |
| `getScanTimeInMillis()` | `Integer` | Group scan period |
| `getOnScriptId()` / `getOffScriptId()` / `getAckScriptId()` | `String` | Script IDs triggered on on/off/ack |
| `getOnNoAckColor()` / `getOnAckColor()` / `getOffNoAckColor()` / `getOffAckColor()` | `String` | Color codes per state |
| `getPrinterIp()` / `getPrinterPort()` | — | Printer address (if any) |
| `getPrintWhenOn()` / `getPrintWhenOff()` / `getPrintWhenAck()` / `getPrintWhenComment()` | `Boolean` | Which events get printed |

## Status Queries

### `ins.getAlarmStatus(name)` / `(projectName, name)`

Returns an `AlarmStatus` enum — two values:

| Value | Meaning |
| --- | --- |
| `"Active"` | Alarm is actively running |
| `"Not Active"` | Alarm is disabled |

```javascript
if (ins.getAlarmStatus("HighTemperature") == "Not Active") {
    ins.notify("info", "Alarm", "HighTemperature is disabled");
}
```

### `ins.getAlarmGroupStatus(name)` / `(projectName, name)`

Same enum for the group.

### `ins.getCurrentAlarmGroupInfo(groupName)`

Returns a summary of the group's current state — `Map<String, Object>` (totals, active count, not-ack'd count, etc.).

```javascript
var info = ins.getCurrentAlarmGroupInfo("Temperature_Alarms");
ins.consoleLog(JSON.stringify(info));
```

## Group Activation / Deactivation

### `ins.activateAlarmGroup(name)` / `(projectName, name)`
### `ins.deactivateAlarmGroup(name)` / `(projectName, name)`

Turns an entire group on/off for maintenance or temporary suspension.

```javascript
// Start maintenance
ins.deactivateAlarmGroup("Temperature_Alarms");
// ...
// End maintenance
ins.activateAlarmGroup("Temperature_Alarms");
```

## Fired Alarm Queries

A "fired alarm" is a single event — from the moment an alarm fires (on) to when it clears (off). Use `includeOff` to decide whether cleared (off) events are included.

### Single Fired Alarm

#### `ins.getFiredAlarm()` / `(projectName)` / `(index)` / `(index, includeOff)`

Returns a single fired alarm by index.

```javascript
var f = ins.getFiredAlarm(0);                 // newest still active
var fAny = ins.getFiredAlarm(0, true);        // newest (including off)
```

### Multiple Fired Alarms

#### `ins.getFiredAlarms(index, count)` / `(index, count, includeOff)` / projectName variants

Returns a window (start = `index`, size = `count`) — `Collection<FiredAlarmDto>`.

```javascript
// Top 10 active fired alarms
var top10 = ins.getFiredAlarms(0, 10);
top10.forEach(function(f) {
    ins.consoleLog(f.getName() + " @ " + f.getOnTime());
});

// Last 50 including off
var last50 = ins.getFiredAlarms(0, 50, true);
```

#### `ins.getFiredAlarmsByDate(startDate, endDate, includeOff, limit)` / projectName variant

Fired alarms in a time range.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 86400000);   // last 24 hours
var history = ins.getFiredAlarmsByDate(start, end, true, 500);
```

#### `ins.getFiredAlarmsByPart(part, page, count)` / projectName variant

Paged list of fired alarms for a specific `part` (partition key).

```javascript
var forLine1 = ins.getFiredAlarmsByPart("Line1", 0, 20);
```

### Current Alarms

#### `ins.getCurrentAlarms(includeOff)` / `(projectName, includeOff)`

Every alarm currently active. `includeOff=true` also includes those that cleared but still need acknowledgement.

```javascript
var active = ins.getCurrentAlarms(false);
ins.setVariableValue("ActiveAlarmCount", { value: active.size() });
```

#### `ins.getCurrentAlarmsByName(alarmNames[], includeOff)`

Returns a map from alarm name to its current fired state — `Map<String, FiredAlarmDto>`.

```javascript
var map = ins.getCurrentAlarmsByName(["HighTemperature", "LowPressure"], false);
if (map.HighTemperature) {
    ins.consoleLog("HighTemperature active, on at " + map.HighTemperature.getOnTime());
}
```

### `FiredAlarmDto` Fields

| Method | Type | Description |
| --- | --- | --- |
| `getName()` / `getDsc()` | `String` | Alarm name and description |
| `getId()` / `getAlarmId()` / `getGroupId()` / `getProjectId()` | `String` | Identifiers |
| `getGroup()` / `getProject()` | `String` | Group and project name |
| `getPart()` | `String` | Partition key |
| `getStatus()` | `FiredAlarmStatus` | `"On"` / `"Off"` |
| `getStatusValue()` | `Integer` | Raw status integer |
| `getFiredAlarmType()` | `FiredAlarmType` | `Digital`, `Custom`, `Analog High High`, `Analog High`, `Analog Low`, `Analog Low Low`, `Analog Set Point` |
| `getOnValue()` / `getOffValue()` / `getOnValueB()` / `getOffValueB()` | `Double` | On/off trigger values |
| `getOnTime()` / `getOnTimeInMs()` | `Date` / `Long` | On timestamp |
| `getOffTime()` / `getOffTimeInMs()` | `Date` / `Long` | Off timestamp (`null` if not yet off) |
| `getAcknowledgeTime()` / `getAcknowledgeTimeInMs()` / `getAcknowledger()` | — | Acknowledgement info |
| `getForcedOff()` / `getForcedOffBy()` | `Boolean` / `String` | Whether forced off |
| `getComment()` / `getCommentedBy()` / `getCommentTime()` / `getCommentTimeInMs()` | — | Comment info |

## Actions on Fired Alarms

Actions (`acknowledgeAlarm`, `forceOffAlarm`, `commentAlarm`) take a **FiredAlarmDto** — typical pattern: fetch the fired alarm first, then operate on it.

### `ins.acknowledgeAlarm(firedAlarmDto)`

```javascript
var active = ins.getCurrentAlarms(false);
active.forEach(function(f) {
    ins.acknowledgeAlarm(f);
});
```

### `ins.forceOffAlarm(firedAlarmDto)`

Forces an alarm off even if the underlying trigger still holds.

```javascript
var f = ins.getFiredAlarm(0);
if (f && f.getName() == "StaleSensor") {
    ins.forceOffAlarm(f);
}
```

### `ins.commentAlarm(firedAlarmDto, comment)`

Attaches an operator comment to the alarm.

```javascript
var f = ins.getFiredAlarm(0);
ins.commentAlarm(f, "Sensor recalibrated — false positive");
```

## Updating an Alarm Definition

### `ins.updateAlarm(alarmName, dto)`

Requires a full `AlarmResponseDto` — use the get/mutate/update pattern.

```javascript
var a = ins.getAlarm("HighTemperature");
a.setDelay(60);                         // delay → 60 s
a.setDsc("High temperature — updated");
ins.updateAlarm("HighTemperature", a);
```

## Example: Alarm Report and Auto-Ack

```javascript
function main() {
    var end = ins.now();
    var start = ins.getDate(end.getTime() - 3600000);

    // Every alarm fired in the last hour
    var events = ins.getFiredAlarmsByDate(start, end, true, 1000);
    ins.writeLog("INFO", "AlarmReport", "Last hour: " + events.size() + " events");

    // After sensor cleanup, auto-ack alarms in the cleanup group
    var active = ins.getCurrentAlarms(false);
    active.forEach(function(f) {
        if (f.getGroup() == "Sensors_Cleanup") {
            ins.commentAlarm(f, "Auto-ack — post-cleanup");
            ins.acknowledgeAlarm(f);
        }
    });
}
main();
```
