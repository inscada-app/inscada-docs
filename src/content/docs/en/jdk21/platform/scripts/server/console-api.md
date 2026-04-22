---
title: "Console API"
description: "Script log output"
sidebar:
  order: 18
---

Console API emits debug or log output while a script runs. Output lands in the platform's script console and the project log.

## `ins.consoleLog(obj)`

Writes the given object (string, number, plain objects, JS primitives) to the console.

```javascript
ins.consoleLog("Debug: started");
ins.consoleLog({ phase: "warm-up", count: 12 });
ins.consoleLog(42);
```

:::tip
There is **no** browser-style `console.log` on the server — there is no stdout. Always use `ins.consoleLog()` for debug output.
:::
