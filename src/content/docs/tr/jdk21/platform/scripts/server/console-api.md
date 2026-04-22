---
title: "Console API"
description: "Script log çıktısı"
sidebar:
  order: 18
---

Console API, script çalışırken debug çıktısı veya log üretir. Çıktı platformun script console'una ve proje log'una düşer.

## `ins.consoleLog(obj)`

Verilen objeyi (string, sayı, objeler, JS primitive'leri) konsola yazar.

```javascript
ins.consoleLog("Debug: başladı");
ins.consoleLog({ phase: "warm-up", count: 12 });
ins.consoleLog(42);
```

:::tip
Script'ten browser-style `console.log` **YOK** — server tarafında stdout yok. Debug çıktısı için her zaman `ins.consoleLog()` kullan.
:::
