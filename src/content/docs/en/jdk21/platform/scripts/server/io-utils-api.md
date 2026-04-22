---
title: "I/O Utils API"
description: "HTTP (REST), network checks, file read/write and JSON serialization"
sidebar:
  order: 16
---

I/O Utils API gathers methods a server-side script needs to talk to external HTTP services, network checks, the file system and JSON. In JDK11 these methods were under Utils; JDK21 splits them into a dedicated sub-API.

## HTTP

### `ins.rest(httpMethod, url, contentType, body)`

Simple HTTP call — sets the `Accept` header to `contentType`.

```javascript
var response = ins.rest("GET",
    "https://jsonplaceholder.typicode.com/todos/1",
    "application/json", null);

// response.statusCode → 200
// response.body       → '{"userId":1,"id":1,"title":"...","completed":false}'
var data = JSON.parse(response.body);
```

### `ins.rest(httpMethod, url, headers, body)`

HTTP call with a custom header map.

```javascript
var headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer xxx"
};

var response = ins.rest("POST",
    "https://api.example.com/events",
    headers,
    { name: "shift_end", ts: Date.now() });
```

### Response Shape

Both overloads of `rest(...)` return a `Map<String, Object>`:

| Field | Type | Description |
| --- | --- | --- |
| `statusCode` | `int` | HTTP status code |
| `body` | `String` | Response body |
| `headers` | `Map<String, List<String>>` | Response headers |

**Supported methods:** `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`.

:::tip
`ins.rest()` runs server-side — the browser's CORS restrictions do not apply.
:::

## Network

### `ins.ping(address, timeout)`

ICMP-style reachability test — returns `Boolean`.

```javascript
var reachable = ins.ping("127.0.0.1", 3000);
if (!reachable) {
    ins.notify("warning", "Network", "PLC is unreachable!");
}
```

### `ins.ping(address, port, timeout)`

Tests whether a specific TCP port is reachable.

```javascript
var portOpen = ins.ping("192.168.1.50", 502, 3000);   // Modbus port
```

## File System

Scripts can only read/write files through these helpers — they operate on the platform's virtual FS.

### `ins.readFile(fileName)`

Returns the file content as `String`.

```javascript
var content = ins.readFile("config.json");
var config = JSON.parse(content);
```

### `ins.readFileAsBytes(fileName)`

Returns the file content as `byte[]`.

```javascript
var bytes = ins.readFileAsBytes("firmware.bin");
// access via bytes.length, bytes[i]
```

### `ins.writeToFile(fileName, text)`

Writes to `fileName` — **overwrites** an existing file (JDK21 has no `append` argument; to append, read first and concatenate).

```javascript
ins.writeToFile("report.csv", "timestamp,temperature,pressure\n");

// Append-like pattern
var current = ins.readFile("report.csv");
ins.writeToFile("report.csv", current + ins.now() + ",25.4,3.2\n");
```

## JSON

### `ins.toJSONStr(object)`

Serializes a JS/Java object to a JSON string.

```javascript
var obj = { temperature: 25.4, status: "ok", tags: ["line1", "ok"] };
var json = ins.toJSONStr(obj);
// '{"temperature":25.4,"status":"ok","tags":["line1","ok"]}'
```

:::note
A script may also use standard `JSON.parse()` / `JSON.stringify()` (built into GraalJS). `ins.toJSONStr()` is specifically useful for serializing Java-side objects correctly.
:::
