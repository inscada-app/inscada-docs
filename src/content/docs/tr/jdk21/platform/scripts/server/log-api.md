---
title: "Log API"
description: "Proje audit log'una kayıt yazma ve sorgulama"
sidebar:
  order: 10
---

Log API; script'in proje audit log'una seviyeli (info / warn / error) kayıt yazmasını ve kayıtları tarih / severity / activity filtresiyle sorgulamasını sağlar. Yazılan loglar platformun Log ekranında ve REST endpoint'lerinde görünür.

## `ins.writeLog(type, activity, msg)` / `(projectName, type, activity, msg)`

Proje audit log'una yeni bir kayıt ekler. `projectName` verilmezse mevcut proje varsayılır.

| Parametre | Tür | Açıklama |
| --- | --- | --- |
| `type` | `String` | Log seviyesi — aşağıya bak |
| `activity` | `String` | İşlem/aktivite etiketi (serbest biçim) |
| `msg` | `String` | Log mesajı |

### Geçerli `type` Değerleri

`type` **küçük harfli** şu üç değerden biri olmalıdır:

| `type` | Severity |
| --- | --- |
| `"error"` | Error |
| `"warn"` | Warning |
| herhangi bir başka değer (örn. `"info"`, boş, `null`) | Information (default) |

```javascript
ins.writeLog("info", "Setpoint Change", "Temperature_C: 48.0 → 55.0");
ins.writeLog("warn", "Network", "Modbus timeout — retry scheduled");
ins.writeLog("error", "BatchProcess", "Checksum mismatch at record 42");
```

:::caution
Büyük harfli `"INFO"` / `"WARNING"` / `"ERROR"` eşleşmez — default'a (INFO) düşer. `"warn"` (4 karakter) doğrudur, `"warning"` değil.
:::

## `ins.getLogsByPage(...)`

Log kayıtlarını sayfalı döner — `Collection<LogEntryDto>`. Üç overload mevcuttur.

### `ins.getLogsByPage(startDate, endDate, page, pageSize)`

En sade hali — mevcut proje, filtre yok.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 3600000);   // son 1 saat

var logs = ins.getLogsByPage(start, end, 0, 50);
```

### `ins.getLogsByPage(activity, logSeverity, startDate, endDate, page, pageSize)`

Activity ve severity filtresi ekler.

```javascript
var errors = ins.getLogsByPage("BatchProcess", "Error", start, end, 0, 100);
errors.forEach(function(e) {
    ins.consoleLog(e.getDttm() + " [" + e.getLogSeverity() + "] " + e.getMsg());
});
```

### `ins.getLogsByPage(projectName, activity, logSeverity, startDate, endDate, page, pageSize)`

Belirli bir projedeki log'lar için.

```javascript
var logs = ins.getLogsByPage("other_project", null, null, start, end, 0, 20);
```

### Filtre Parametreleri

| Parametre | Açıklama |
| --- | --- |
| `activity` | Tam eşleşme; tüm activity'ler için `null` ver |
| `logSeverity` | `"Information"`, `"Warning"`, `"Error"` — **tam değer** (writeLog'dan farklı); tümü için `null` |

### `LogEntryDto` Alanları

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getActivity()` | `String` | Activity etiketi |
| `getMsg()` | `String` | Log mesajı |
| `getLogSeverity()` | `LogSeverity` | `"Information"` / `"Warning"` / `"Error"` |
| `getDttm()` | `Date` | Log zamanı |
| `getTime()` | `Long` | Epoch ms |
| `getProject()` / `getProjectId()` | `String` | Proje adı / ID |

## Örnek: Son 24 Saatteki Error Sayısını Raporla

```javascript
function main() {
    var end = ins.now();
    var start = ins.getDate(end.getTime() - 86400000);

    var errors = ins.getLogsByPage(null, "Error", start, end, 0, 1000);
    ins.setVariableValue("ErrorCount_24h", { value: errors.size() });

    if (errors.size() > 50) {
        ins.notify("error", "Log alarmı", "Son 24 saatte " + errors.size() + " hata");
    }
}
main();
```
