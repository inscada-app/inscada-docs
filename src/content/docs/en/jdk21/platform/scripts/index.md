---
title: "Script Engine"
description: "inSCADA JDK21 script engine — overview of the server-side ins.* and client-side Inscada.* APIs"
sidebar:
  order: 0
  label: "Overview"
---

inSCADA JDK21 ships two distinct scripting environments: **server-side** (`ins.*`) and **client-side** (`Inscada.*`). They run in different places, target different use cases, and expose isolated API surfaces.

## Two Separate Worlds

|  | Server-side (`ins.*`) | Client-side (`Inscada.*`) |
| --- | --- | --- |
| **Where it runs** | inSCADA server (JVM) | Browser (browser JS) |
| **Engine** | GraalJS (Nashorn-compat) | The browser's own engine |
| **Binding** | `ins` (global) | `new InscadaApi(projectName)` |
| **Purpose** | Automation, calculation, alarms, scheduled work | User interaction, UI control |
| **Typical usage** | Scheduled scripts, variable / alarm / log expressions, MQTT parsing | Animation click, custom HTML widgets, dashboard widgets |

→ [Server-side (ins.*) details](/docs/en/jdk21/platform/scripts/server/)
→ [Client-side (Inscada.*) details](/docs/en/jdk21/platform/scripts/client/)

## Where Scripts Run

Scripts fire in many places across the platform. The table below shows the environment each one runs in.

| Context | Environment | Trigger |
| --- | --- | --- |
| **Scheduled Script** | server (`ins.*`) | Periodic / Daily / Once / manual |
| **Variable Expression** | server (`ins.*`) | Automatically on poll cycle |
| **Alarm Expression** | server (`ins.*`) | Automatically on value change |
| **Log Expression** | server (`ins.*`) | Automatically on poll cycle |
| **MQTT Subscribe/Publish** | server (`ins.*`) | On every MQTT message |
| **Animation Expression** | server (`ins.*`) | On UI update (remote) |
| **Animation Click Script** | client (`Inscada.*`) | User click |
| **Custom HTML Widget** | client (`Inscada.*`, sandboxed iframe) | Inside the widget |
| **Dashboard Widget** | client (`Inscada.*`) | Dashboard render / interaction |

:::tip[Which one should I pick?]
Reading/writing values, firing alarms, doing scheduled calculations — **server-side** (`ins.*`). Showing a popup to the user, SVG zoom, swapping animations — **client-side** (`Inscada.*`). To combine them: the client's mirror methods call their server-side counterparts asynchronously.
:::

## Sandboxing and Security

Scripts run in isolation in both environments, but the constraints differ:

- **Server side** — `HostAccess.EXPLICIT`: only Java methods annotated `@HostAccess.Export` are accessible. `eval` and `with` are disabled. No threads, native access, process creation, or direct file I/O. Default limits: 100k statements, 60s timeout.
- **Client side** — custom HTML widgets run inside a sandboxed iframe; they reach the host page API through `InscadaBridge` using origin-validated `postMessage`.

:::caution
Scripts run inside the platform. Infinite loops or heavy computation impact performance. Server-side enforces 100k statements / 60s as defaults; on the client avoid code that freezes the browser tab.
:::
