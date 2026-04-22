---
title: "Script API"
description: "Script zamanlama, manuel çalıştırma, durum sorgulama ve script'ler arası paylaşımlı global object"
sidebar:
  order: 7
---

Script API; diğer script'leri zamanlayıcıya bağlama / çıkarma, anında çalıştırma, durum sorgulama ve script'ler arası veri paylaşmak için kullanılır. Paylaşımlı veri katmanı Redis üzerinden tutulur ve proje-scoped'tur.

## Script Meta Verisi ve Durum

### `ins.getScript(name)` / `(projectName, name)`

Bir script'in tanımını döner — `RepeatableScriptResponseDto`.

```javascript
var s = ins.getScript("HourlyReport");
ins.consoleLog(s.getName() + " type=" + s.getType() + " period=" + s.getPeriod() + "ms");
```

Alanlar:

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` / `getDsc()` | `String` | Script adı ve açıklama |
| `getCode()` | `String` | JavaScript kaynak kodu |
| `getType()` | `ScheduleType` | `PERIODIC` / `DAILY` / `ONCE` / `NONE` |
| `getPeriod()` | `Integer` | Periodic tipi için ms cinsinden periyot |
| `getOffset()` | `Integer` | Periodic için ms cinsinden offset |
| `getDelay()` | `Integer` | Once tipi için gecikme (ms) |
| `getTime()` | `Date` | Daily için tetikleme saati |
| `getLog()` | `Boolean` | Çalıştırma log'u aktif mi |
| `getProjectId()` | `String` | Proje ID |

### `ins.getScriptStatus(name)` / `(projectName, name)`

`ScriptStatus` enum döner — iki değer:

| Değer | Anlam |
| --- | --- |
| `"Scheduled"` | Zamanlayıcıya bağlı |
| `"Not Scheduled"` | Bağlı değil (manuel çalıştırılabilir) |

```javascript
if (ins.getScriptStatus("HourlyReport") == "Not Scheduled") {
    ins.scheduleScript("HourlyReport");
}
```

## Script Yönetimi

### `ins.scheduleScript(name)` / `(projectName, name)`

Script'i tanımındaki `ScheduleType`'a göre zamanlayıcıya ekler. `type=NONE` olan script'lere etkisi yoktur.

```javascript
ins.scheduleScript("HourlyReport");
```

### `ins.cancelScript(name)` / `(projectName, name)`

Script'i **zamanlayıcıdan çıkarır** — kalan tetiklemeler iptal edilir, ama o sırada çalışan bir execution kesilmez (mevcut execution doğal sonuna kadar gider).

```javascript
ins.cancelScript("HourlyReport");
```

### `ins.executeScript(name)` / `(projectName, name)`

Script'i **anında** çalıştırır ve döndürdüğü değeri döner. Senkron çalışır — script bitene kadar blocker.

```javascript
var dailyTotal = ins.executeScript("Calculate_DailyTotal");
ins.setVariableValue("DailyProduction_kWh", { value: dailyTotal });
```

:::note
Çağıran ve çağrılan script aynı statement/timeout limitleri altındadır; çağrılan script'in çalışma süresi, çağıranın toplam süresine yazılır. Script yoksa `ScriptException` fırlar.
:::

## Global Object — Script'ler Arası Paylaşım

Script'ler birbirinden izole çalışır (paylaşımlı memory yoktur). Bunu aşmak için `setGlobalObject` / `getGlobalObject` ikilisi Redis tabanlı bir anahtar-değer deposu sunar. Anahtarlar proje-scoped'tır (`project:<projectId>:global-object:<name>`).

### `ins.setGlobalObject(name, object)`

Objeyi **süresiz** (TTL yok) saklar.

```javascript
ins.setGlobalObject("daily_counter", 42);
ins.setGlobalObject("shift_data", {
    shift: "A",
    count: 150,
    startTime: ins.now().toString()
});
```

### `ins.setGlobalObject(name, object, ms)`

Objeyi **TTL ile** saklar — `ms` milisaniye sonra otomatik silinir (Redis `SET PX`).

```javascript
// 60 saniye sonra silinecek cache
ins.setGlobalObject("temp_cache", { value: 99 }, 60000);
```

### `ins.getGlobalObject(name)`

Objeyi döner; yoksa `null`. TTL'e dokunmaz.

```javascript
var counter = ins.getGlobalObject("daily_counter");
// → 42 veya null
```

### `ins.getGlobalObject(name, ms)`

Objeyi döner **ve aynı anda TTL'i `ms`'e resetler** (sliding TTL — Redis `PEXPIRE`). Tipik kullanım: "Her okuma objeyi canlı tutsun."

```javascript
// Oturum gibi davranan veri: her okuma 5 dk canlılık ekler
var session = ins.getGlobalObject("user_session", 5 * 60 * 1000);
```

:::note
Objeler JSON olarak serileşir. Date → string'e dönüşür, fonksiyonlar / class instance'ları kaybolur. Yalnızca plain objeler, diziler, sayılar, string'ler, boolean'lar güvenlidir.
:::

:::note
Redis paylaşımı sayesinde global object'ler **cluster ortamında** tüm node'lardan görünürdür; restart'ta Redis kalıcılığına göre davranır.
:::

## Kullanım Kalıpları

### Backend → UI Veri Aktarımı

```javascript
// Backend script (her 10 sn):
function main() {
    ins.setGlobalObject("dashboard_summary", {
        power:   ins.getVariableValue("ActivePower_kW").value,
        voltage: ins.getVariableValue("Voltage_V").value,
        updatedAt: ins.now().getTime()
    }, 30000);   // 30 sn TTL — veri "stale" olmasın
}
main();
```

UI tarafındaki custom HTML widget, client-side `Inscada.*` ile aynı anahtara okuma yapabilir.

### Rate Limiting

```javascript
function main() {
    var last = ins.getGlobalObject("report_last_run");
    var now  = ins.now().getTime();

    if (last && (now - last) < 3600000) {
        ins.consoleLog("1 saat dolmadı, atla");
        return;
    }

    ins.executeScript("generate_hourly_report");
    ins.setGlobalObject("report_last_run", now);
}
main();
```

### Bayrak / Mutex

```javascript
function main() {
    if (ins.getGlobalObject("maintenance_mode")) {
        ins.consoleLog("Bakım modu, script atlandı");
        return;
    }
    // normal iş...
}
main();
```
