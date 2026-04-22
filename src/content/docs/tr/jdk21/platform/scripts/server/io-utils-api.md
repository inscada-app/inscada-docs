---
title: "I/O Utils API"
description: "HTTP (REST), ağ testi, dosya okuma/yazma ve JSON serialize"
sidebar:
  order: 16
---

I/O Utils API, script'in sunucu tarafından dış HTTP servisleri, ağ testleri, dosya sistemi ve JSON ile etkileşimi için gerekli metodları toplar. JDK11'de Utils altında olan bu metodlar JDK21'de ayrı bir alt API'ye alındı.

## HTTP

### `ins.rest(httpMethod, url, contentType, body)`

Basit HTTP çağrısı — `Accept` header'ı `contentType` olarak ayarlanır.

```javascript
var response = ins.rest("GET",
    "https://jsonplaceholder.typicode.com/todos/1",
    "application/json", null);

// response.statusCode → 200
// response.body       → '{"userId":1,"id":1,"title":"...","completed":false}'
var data = JSON.parse(response.body);
```

### `ins.rest(httpMethod, url, headers, body)`

Özel header map'i ile HTTP çağrısı.

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

### Yanıt Şekli

`rest(...)` her iki overload da bir `Map<String, Object>` döner:

| Alan | Tür | Açıklama |
| --- | --- | --- |
| `statusCode` | `int` | HTTP durum kodu |
| `body` | `String` | Yanıt gövdesi |
| `headers` | `Map<String, List<String>>` | Yanıt header'ları |

**Desteklenen metodlar:** `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`.

:::tip
`ins.rest()` sunucu tarafında çalışır — tarayıcıdaki CORS kısıtları geçerli değildir.
:::

## Ağ

### `ins.ping(address, timeout)`

ICMP benzeri erişilebilirlik testi — `Boolean` döner.

```javascript
var reachable = ins.ping("127.0.0.1", 3000);
if (!reachable) {
    ins.notify("warning", "Ağ", "PLC erişilemez!");
}
```

### `ins.ping(address, port, timeout)`

Belirli bir TCP portuna bağlanabilme testi.

```javascript
var portOpen = ins.ping("192.168.1.50", 502, 3000);   // Modbus portu
```

## Dosya Sistemi

Script'ler yalnızca bu metodlar üzerinden dosyaya erişebilir — platform sanal FS'sine yazar/okur.

### `ins.readFile(fileName)`

Dosya içeriğini `String` olarak döner.

```javascript
var content = ins.readFile("config.json");
var config = JSON.parse(content);
```

### `ins.readFileAsBytes(fileName)`

Dosya içeriğini `byte[]` olarak döner.

```javascript
var bytes = ins.readFileAsBytes("firmware.bin");
// bytes.length, bytes[i] gibi erişim
```

### `ins.writeToFile(fileName, text)`

`fileName`'e yazar — dosya varsa **üzerine yazar** (JDK21'de `append` parametresi yoktur; append istiyorsan önce okuyup birleştirmelisin).

```javascript
ins.writeToFile("report.csv", "timestamp,temperature,pressure\n");

// Append benzeri davranış
var current = ins.readFile("report.csv");
ins.writeToFile("report.csv", current + ins.now() + ",25.4,3.2\n");
```

## JSON

### `ins.toJSONStr(object)`

JS/Java objesini JSON string'ine serialize eder.

```javascript
var obj = { temperature: 25.4, status: "ok", tags: ["line1", "ok"] };
var json = ins.toJSONStr(obj);
// '{"temperature":25.4,"status":"ok","tags":["line1","ok"]}'
```

:::note
Script'in kendisi standart `JSON.parse()` ve `JSON.stringify()`'ı da kullanabilir (GraalJS yerleşik). `ins.toJSONStr()` özellikle Java-tarafı objelerini doğru şekilde serialize etmek için vardır.
:::
