---
title: "Connection API"
description: "Start/stop/query communication connections and update connection, device and frame configuration"
sidebar:
  order: 2
---

Connection API manages communication connections (Modbus, OPC-UA, IEC-104, S7, MQTT, BACnet, etc.) from scripts and reads/updates the **Connection → Device → Frame** hierarchy.

## Connection Control

### `ins.startConnection(name)` / `ins.startConnection(projectName, name)`

Starts the connection. `projectName` defaults to the current project if omitted.

```javascript
ins.startConnection("MODBUS-PLC");
ins.startConnection("otherProject", "MODBUS-PLC");
```

### `ins.stopConnection(name)` / `ins.stopConnection(projectName, name)`

Stops the connection.

```javascript
ins.stopConnection("MODBUS-PLC");
```

### `ins.getConnectionStatus(name)` / `ins.getConnectionStatus(projectName, name)`

Returns a `ConnectionStatus` enum — only two values exist:

| Value | Meaning |
| --- | --- |
| `"Connected"` | Connection is active |
| `"Disconnected"` | Connection is closed or lost |

```javascript
var status = ins.getConnectionStatus("MODBUS-PLC");
if (status == "Disconnected") {
    ins.notify("warning", "Connection", "MODBUS-PLC is not connected");
}
```

## Fetching Connection, Device, Frame Info

### `ins.getConnection(name)` / `ins.getConnection(projectName, name)`

Returns a `ConnectionResponseDto`.

Fields:

| Method | Type | Description |
| --- | --- | --- |
| `getName()` | `String` | Connection name |
| `getDsc()` | `String` | Description |
| `getProjectId()` | `String` | Project ID |
| `getProtocol()` | `Protocol` | Protocol (MODBUS, OPC_UA, S7, …) |
| `getIp()` | `String` | IP address |
| `getPort()` | `Integer` | Port |
| `getConfig()` | `Map<String, Object>` | Protocol-specific extra settings |

```javascript
var c = ins.getConnection("MODBUS-PLC");
ins.consoleLog(c.getProtocol() + " " + c.getIp() + ":" + c.getPort());
```

### `ins.getDevice(connectionName, deviceName)`

Returns a `DeviceResponseDto`.

| Method | Type | Description |
| --- | --- | --- |
| `getName()` | `String` | Device name |
| `getDsc()` | `String` | Description |
| `getConnectionId()` | `String` | Parent connection ID |
| `getProtocol()` | `Protocol` | Protocol |
| `getConfig()` | `Map<String, Object>` | Device-specific settings (slave ID, node ID, …) |

```javascript
var d = ins.getDevice("MODBUS-PLC", "Device1");
ins.consoleLog("Slave: " + d.getConfig().slaveId);
```

### `ins.getFrame(connectionName, deviceName, frameName)`

Returns a `FrameResponseDto`.

| Method | Type | Description |
| --- | --- | --- |
| `getName()` | `String` | Frame name |
| `getDsc()` | `String` | Description |
| `getDeviceId()` | `String` | Parent device ID |
| `getProtocol()` | `Protocol` | Protocol |
| `getMinutesOffset()` | `Integer` | Minute offset (for scan windows) |
| `getScanTimeFactor()` | `Integer` | Scan time multiplier |
| `getIsReadable()` | `Boolean` | Read enabled |
| `getIsWritable()` | `Boolean` | Write enabled |
| `getConfig()` | `Map<String, Object>` | Frame-specific (e.g. Modbus address range) |

```javascript
var f = ins.getFrame("MODBUS-PLC", "Device1", "HoldingRegs_0_100");
ins.consoleLog("Scan x" + f.getScanTimeFactor() + " — offset " + f.getMinutesOffset() + "m");
```

## Configuration Updates

Update methods require a **complete DTO** — the typical pattern is: read with `getConnection` / `getDevice` / `getFrame`, mutate the field, write back with `updateX`.

### `ins.updateConnection(connectionName, dto)`

```javascript
var c = ins.getConnection("MODBUS-PLC");
c.setIp("192.168.1.100");
c.setPort(502);
ins.updateConnection("MODBUS-PLC", c);
```

### `ins.updateDevice(connectionName, deviceName, dto)`

```javascript
var d = ins.getDevice("MODBUS-PLC", "Device1");
d.setDsc("Main switchgear — updated");
ins.updateDevice("MODBUS-PLC", "Device1", d);
```

### `ins.updateFrame(connectionName, deviceName, frameName, dto)`

```javascript
var f = ins.getFrame("MODBUS-PLC", "Device1", "HoldingRegs_0_100");
f.setScanTimeFactor(5);           // sparser scan interval
f.setIsWritable(false);            // make read-only
ins.updateFrame("MODBUS-PLC", "Device1", "HoldingRegs_0_100", f);
```

:::caution
Update calls can affect a running connection — in busy field scenarios `stopConnection` first, then update, then `startConnection` is a safer sequence.
:::

## Example: Refresh Connection on IP Change

```javascript
function main() {
    var c = ins.getConnection("MODBUS-PLC");
    if (c.getIp() == "192.168.1.100") {
        ins.consoleLog("IP already up to date");
        return;
    }

    ins.stopConnection("MODBUS-PLC");
    c.setIp("192.168.1.100");
    ins.updateConnection("MODBUS-PLC", c);
    ins.startConnection("MODBUS-PLC");
    ins.writeLog("INFO", "Connection", "MODBUS-PLC → 192.168.1.100");
}
main();
```
