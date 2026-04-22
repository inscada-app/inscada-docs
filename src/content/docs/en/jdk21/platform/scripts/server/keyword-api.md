---
title: "Keyword API"
description: "Record metadata (keywords) from a script"
sidebar:
  order: 15
---

Keyword API lets a running script persist free-form metadata (keywords). Typical use: persisting the outcome of a batch, shift, or calibration cycle as key-value data and surfacing it later in reports or search.

## `ins.saveKeyword(objectMap)`

Writes a key-value map to the platform's keyword store (`Map<String, Object>`).

```javascript
ins.saveKeyword({
    "batch_id": "B-240423-A",
    "operator": user.name,
    "end_temperature": 85.4,
    "duration_sec": 1245,
    "status": "ok"
});
```

Values may be primitives (number, string, boolean) or nested objects/arrays.
