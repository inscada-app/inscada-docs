---
title: "System API"
description: "Shutdown / restart, OS command execution, system-time setting and pending system requests"
sidebar:
  order: 12
---

System API runs OS-level operations on the host the platform is running on: shut down / restart the platform, set the system clock, run arbitrary commands, and manage pending system requests.

:::caution
Every call requires the **`EXEC_SYSTEM_COMMAND`** authority. Misuse can stop the server, corrupt the clock, or turn into arbitrary code execution. Each call is audited in the project log.
:::

## `ins.shutdown()`

Shuts the platform process down.

```javascript
ins.shutdown();
```

## `ins.restart()`

Restarts the platform.

```javascript
var h = ins.now().getHours();
if (h >= 2 && h <= 4) {
    ins.writeLog("warn", "System", "Scheduled restart");
    ins.restart();
}
```

## `ins.setDateTime(ms, dateCmdFormat)`

Sets the server's system clock to `ms` (epoch).

| Parameter | Description |
| --- | --- |
| `ms` | Target time (epoch milliseconds) |
| `dateCmdFormat` | Java `SimpleDateFormat` pattern used **only on Windows** (e.g. `"MM-dd-yyyy"`) — it formats `ms` and passes the result to `cmd /c date <value>`; the time is set separately through `time` with `HH:mm:ss`. **On Linux this parameter is ignored** — the service internally uses a fixed `yyyy-MM-dd HH:mm:ss` format plus `date -s`. |

```javascript
// Windows
ins.setDateTime(Date.now(), "MM-dd-yyyy");

// Linux — format is ignored, may be empty
ins.setDateTime(Date.now(), "");
```

## `ins.exec(command)` — Two Overloads

Runs an OS command on the host. The return is **not** the command's output — it is the **exit code** (int). `0` means success.

### `ins.exec(String[] command)` *(recommended)*

Pass the argument list as an array — no shell parsing, lower injection risk.

```javascript
var rc = ins.exec(["df", "-h", "/"]);
if (rc != 0) {
    ins.writeLog("error", "System", "df failed — exit " + rc);
}
```

### `ins.exec(String commandLine)`

Pass a single string; it is split on whitespace internally (simple splitter, does not honor quoted arguments).

```javascript
ins.exec("df -h /");
```

:::caution
If you need the command's **output**, `exec` is not enough. Redirect the command to a file (`> /opt/inscada/tmp/out.txt`) and read it back with `ins.readFile("tmp/out.txt")`. `exec` only returns the exit code.
:::

:::caution
**Never** pass user input directly to `exec` — it can be escalated into arbitrary command execution. Validate / whitelist first.
:::

## System Request Queue

The platform maintains a queue of "system requests" (e.g. user-approved shutdown / restart). This API gives scripts access to that queue.

### `ins.getSystemRequests()`

Returns pending system requests — `Collection<SystemRequestDto>`.

```javascript
var reqs = ins.getSystemRequests();
reqs.forEach(function(r) {
    ins.consoleLog(r.getType() + " — " + r.getRequestDate());
});
```

### `ins.deleteSystemRequest(systemRequest)`

Removes a request from the queue.

```javascript
var reqs = ins.getSystemRequests();
reqs.forEach(function(r) {
    if (r.getType() == "RESTART") ins.deleteSystemRequest(r);
});
```

### `SystemRequestDto` Fields

| Method | Type | Description |
| --- | --- | --- |
| `getType()` | `String` | Request type (e.g. `"SHUTDOWN"`, `"RESTART"`) |
| `getRequester()` | `Map<String, Object>` | Requesting-user info |
| `getRequestDate()` | `Date` | Creation time |

## Example: Alert on Low Disk Space

```javascript
function main() {
    var rc = ins.exec(["sh", "-c", "df -h / | awk 'NR==2 {print $5}' | tr -d '%' > /opt/inscada/tmp/disk.txt"]);
    if (rc != 0) return;

    var used = parseInt(ins.readFile("tmp/disk.txt").trim(), 10);
    if (used > 95) {
        ins.sendMail(["ops"], "Disk " + used + "%", "Critical disk usage — restart planned in next maintenance window");
    }
}
main();
```
