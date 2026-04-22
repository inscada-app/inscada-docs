---
title: "Server-Side Script Engine"
description: "ins.* — GraalJS-based server-side script engine with Nashorn compatibility"
sidebar:
  order: 0
  label: "Overview"
---

Server-side scripts run on the inSCADA server and reach every platform capability through the `ins` global object. This page covers the engine, sandbox, scheduling and lifecycle; individual sub-APIs have their own pages.

## Engine

JDK21 replaced Nashorn with **GraalJS** (GraalVM JavaScript). To keep existing JDK11 scripts running unchanged, **Nashorn compatibility mode** is enabled (`js.nashorn-compat: true`). In practice this means:

- Your existing JDK11 scripts (only `var`, `function`, ES5) run without any change.
- You may use **modern JS** if you want: `let` / `const`, arrow functions, template literals, destructuring, classes, spread / rest, `async` / `await`.
- Mixing both styles in the same script is safe.

### Supported JavaScript Features

| Feature | Status |
| --- | --- |
| `var`, `function`, ES5 | ✓ |
| `let`, `const` | ✓ |
| Arrow functions (`=>`) | ✓ |
| Template literals (backtick) | ✓ |
| Destructuring | ✓ |
| `class` / `extends` | ✓ |
| Spread / rest (`...`) | ✓ |
| `async` / `await` | ✓ |
| `eval(...)` | ✗ (disabled by sandbox) |
| `with (...)` | ✗ (disabled by sandbox) |

## Sandbox Restrictions

Script security is enforced through Java host access. A script may only call Java methods marked `@HostAccess.Export` — `ins.*` is the surface exposed that way.

### What's Not Allowed

| Blocked | Reason |
| --- | --- |
| Creating threads | Scripts run in a shared thread pool |
| Direct file system access | Use `ins.readFile()` / `ins.writeToFile()` instead |
| Native code (JNI / JNA) | — |
| Environment variable access | — |
| Polyglot (other languages) | — |
| `eval`, `with` | — |

System-level operations such as `exec`, `shutdown`, `restart` are only available through `ins.*` and are recorded to the project log.

### Resource Limits

| Limit | Default | Config key |
| --- | --- | --- |
| Statement count | 100 000 | `ins.script.maxStatementCount` |
| Execution timeout | 60 seconds | `ins.script.execution-timeout` |

A script that exceeds the limits is cancelled and ends with `ScriptException`; the error is logged, the platform is unaffected.

## Global Bindings

Objects automatically injected into the script context:

| Global | Type | Description |
| --- | --- | --- |
| `ins` | `InscadaApi` | Union of every sub-API the platform exposes (list below) |
| `user` | `CurrentUserBinding` | Current user info — `id`, `name`, `roles`, `permissions`, `activeSpace`, `remoteAddress`, `spaces`, `menus` |
| Custom bindings | optional | Additional context objects provided via a script definition's `bindings` field |

Nashorn's meta globals (`quit`, `exit`, `print`, `load`, `loadWithNewGlobal`, `$ARG`, `$ENV`, `$EXEC`, `$OPTIONS`, `$OUT`, `$ERR`, `$EXIT`) are stubbed to no-op for safety — calling them does nothing.

## Scheduled Scripts

Standalone automation jobs are defined from **Development → Scripts**.

### Schedule Types

| Type | Parameters | Description |
| --- | --- | --- |
| **Periodic** | Period (ms), Offset (ms) | Repeat at a fixed interval |
| **Daily** | Hour:Minute | Every day at a specific time |
| **Once** | Delay (ms) | One-shot execution |
| **None** | — | No automatic trigger — invoked manually via `ins.executeScript()` or REST |

### Script Definition Fields

| Field | Description |
| --- | --- |
| **Name** | Unique script name |
| **Code** | JavaScript source |
| **Schedule Type** | `Periodic` / `Daily` / `Once` / `None` |
| **Period / Time** | Schedule parameter |
| **Log** | Record execution logs |
| **Compile** | Compiled-source cache enabled (recommended: on) |
| **Bindings** | Optional extra context bindings |

### Script over REST

```json
{
  "id": 159,
  "name": "Chart_ActiveReactivePower",
  "projectId": 153,
  "type": "None",
  "log": false,
  "compile": true,
  "owner": "inscada",
  "code": "function main() { /* ... */ } main();"
}
```

## Lifecycle

1. **Compilation** — the code is SHA-256 hashed and written to a Caffeine cache (default 2000 entries, 60 min idle TTL). The same code is not recompiled on subsequent runs.
2. **Execution** — runs on the `inSCADA-script-executor-*` daemon thread pool (core = CPU count, max = 8 × CPU, queue = 2048).
3. **Result** — the return value is unwrapped from a GraalVM `Value` to a Java object — `Map<String, Object>` / `List` / primitive.
4. **Error / Timeout** — if the timeout is exceeded the task is cancelled via `Future.cancel(true)`; the error is recorded to the project log as `ScriptException`.

## Example

The same task in ES5 and modern JS — both are valid:

```javascript
// ES5 style (compatible with JDK11 scripts)
function main() {
    var active = ins.getVariableValue("ActivePower_kW");
    var reactive = ins.getVariableValue("ReactivePower_kVAr");
    var s = Math.sqrt(active.value * active.value + reactive.value * reactive.value);
    ins.setVariableValue("ApparentPower_kVA", { value: s });
    ins.writeLog("INFO", "PowerCalc", "S = " + s.toFixed(2) + " kVA");
}
main();
```

```javascript
// Modern JS style (optional in JDK21)
function main() {
    const active = ins.getVariableValue("ActivePower_kW");
    const reactive = ins.getVariableValue("ReactivePower_kVAr");
    const s = Math.sqrt(active.value ** 2 + reactive.value ** 2);
    ins.setVariableValue("ApparentPower_kVA", { value: s });
    ins.writeLog("INFO", "PowerCalc", `S = ${s.toFixed(2)} kVA`);
}
main();
```

## Sub-APIs

| Module | Short |
| --- | --- |
| [Variable API](/docs/en/jdk21/platform/scripts/server/variable-api/) | Read/write variables, stats |
| [Connection API](/docs/en/jdk21/platform/scripts/server/connection-api/) | Connection / device / frame |
| [Alarm API](/docs/en/jdk21/platform/scripts/server/alarm-api/) | Alarm state, fired alarm history |
| [Trend API](/docs/en/jdk21/platform/scripts/server/trend-api/) | Trend tag management |
| [Datasource API](/docs/en/jdk21/platform/scripts/server/datasource-api/) | SQL, InfluxQL |
| [Data Transfer API](/docs/en/jdk21/platform/scripts/server/datatransfer-api/) | File-based data transfer |
| [Notification API](/docs/en/jdk21/platform/scripts/server/notification-api/) | Mail, SMS, web notification |
| [Script API](/docs/en/jdk21/platform/scripts/server/script-api/) | Script scheduling, global object |
| [Report API](/docs/en/jdk21/platform/scripts/server/report-api/) | Classic + Jasper reports |
| [Project API](/docs/en/jdk21/platform/scripts/server/project-api/) | Project info, location |
| [Log API](/docs/en/jdk21/platform/scripts/server/log-api/) | Audit log |
| [System API](/docs/en/jdk21/platform/scripts/server/system-api/) | Shutdown, restart, exec |
| [User API](/docs/en/jdk21/platform/scripts/server/user-api/) | User listing |
| [I/O Utils API](/docs/en/jdk21/platform/scripts/server/io-utils-api/) | REST, ping, file, JSON |
| [Utils API](/docs/en/jdk21/platform/scripts/server/utils-api/) | uuid, date, bit ops, formatting |
| [Language API](/docs/en/jdk21/platform/scripts/server/language-api/) | `loc` |
| [Keyword API](/docs/en/jdk21/platform/scripts/server/keyword-api/) | Metadata |
| [Console API](/docs/en/jdk21/platform/scripts/server/console-api/) | `consoleLog` |
