---
title: "Variable API"
description: "Anlık ve tarihsel değişken değerleri, bulk okuma/yazma, istatistikler ve değişken meta verileri"
sidebar:
  order: 1
---

Variable API, script'ler içinden inSCADA değişkenlerinin **anlık** ve **tarihsel** değerlerine erişir, toplu okuma/yazma, istatistik sorgulama ve değişken metadata yönetimi sunar.

## Anlık Değer Okuma

### `ins.getVariableValue(name)` / `ins.getVariableValue(projectName, name)`

Bir değişkenin anlık değerini `VariableValueDto` olarak döner. Veri in-memory cache'tendir (< 1 ms).

```javascript
var v = ins.getVariableValue("ActivePower_kW");
var power = v.value;          // veya v.getValue()
var ts = v.dateInMs;          // epoch ms
```

Farklı projeden okuma:

```javascript
var v = ins.getVariableValue("other_project", "pressure");
```

### `ins.getVariableValue(name, index)` / `ins.getVariableValue(projectName, name, index)`

Dizi (array) değişkenin belirtilen index'indeki elemanı döner.

```javascript
var el3 = ins.getVariableValue("measurements_array", 3);   // 4. eleman
```

### `ins.getVariableValues(name, fromIndex, toIndex)` / `(projectName, name, fromIndex, toIndex)`

Dizi değişkenin bir aralığını (`fromIndex`–`toIndex` dahil) toplu döner — `Collection<VariableValueDto>`.

```javascript
var range = ins.getVariableValues("measurements_array", 0, 9);   // ilk 10 eleman
```

### `ins.getVariableValues(names[])` / `(projectName, names[])`

Birden fazla değişkeni tek çağrıda okur — `Map<String, VariableValueDto>` döner. Tek tek okumaktan çok daha hızlıdır.

```javascript
var vals = ins.getVariableValues(["ActivePower_kW", "Voltage_V", "Current_A"]);
var p = vals.ActivePower_kW.value;
var u = vals.Voltage_V.value;
var i = vals.Current_A.value;
```

### `ins.getProjectVariableValues()` / `(projectName)`

Projedeki **tüm** değişkenlerin anlık değerlerini döner — `Map<String, VariableValueDto>`.

```javascript
var all = ins.getProjectVariableValues();
Object.keys(all).forEach(function(name) {
    ins.consoleLog(name + " = " + all[name].value);
});
```

### `VariableValueDto` Alanları

| Alan / Metod | Tür | Açıklama |
| --- | --- | --- |
| `getValue()` / `.value` | `Object` | Ölçeklenmiş (engineering) değer |
| `getDate()` / `.date` | `Date` | Değer zaman damgası |
| `getDateInMs()` / `.dateInMs` | `Long` | Aynı zaman damgası — epoch ms |
| `getDttm()` / `.dttm` | `Date` | Sunucu alınış zamanı |
| `getTime()` / `.time` | `Long` | Sunucu alınış zamanı — epoch ms |
| `getVariableShortInfo()` / `.variableShortInfo` | `VariableShortInfoDto` | Değişken metadata: `name`, `dsc`, `connection`, `device`, `frame`, `project` |

:::note
`flags` (örn. `scaled`) ve `extras` (örn. `raw_value`) alanları script'e `@HostAccess.Export` ile açılmamıştır — bunlara doğrudan `v.flags` / `v.extras` ile erişilemez; yalnızca JSON serileşmesinde (`ins.toJSONStr(v)` veya REST yanıtında) görünürler.
:::

## Değer Yazma

### `ins.setVariableValue(name, details)` / `(projectName, name, details)`

Bir değişkene yazar. `details` — zorunlu `value` anahtarı içeren bir `Map<String, Object>`.

```javascript
ins.setVariableValue("Temperature_C", { value: 55.0 });
ins.setVariableValue("GridStatus", { value: true });
ins.setVariableValue("other_project", "target_temp", { value: 80.0 });
```

### `ins.setVariableValues(map)` / `(projectName, map)`

Toplu yazım — `Map<String, Map<String, Object>>`.

```javascript
ins.setVariableValues({
    "Temperature_C": { value: 42.5 },
    "Voltage_V": { value: 228.0 },
    "PumpRun": { value: true }
});
```

### `ins.mapVariableValue(src, dest)` / `(src, dest, defaultValue)` / projectName varyantları

Kaynak değişkenin anlık değerini hedef değişkene kopyalar. Kaynak `null`/okunmamışsa opsiyonel `defaultValue` kullanılır.

```javascript
ins.mapVariableValue("Temperature_C", "display_temp");
ins.mapVariableValue("Temperature_C", "display_temp", 0);
```

### `ins.toggleVariableValue(name)` / `(projectName, name)`

Boolean değişkeni tersler (`true` ↔ `false`).

```javascript
ins.toggleVariableValue("GridStatus");
```

## Tarihsel Veri (Logged Values)

### `ins.getLoggedVariableValuesByPage(names[], startDate, endDate, page, pageSize)`

Belirli aralıktaki log kayıtlarını sayfalı olarak döner — `Collection<LoggedVariableValueDto>`. Sonuçlar **yeniden eskiye** (DESC) sıralıdır.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 300000);   // son 5 dakika

var logs = ins.getLoggedVariableValuesByPage(
    ["ActivePower_kW"],
    start, end,
    0,    // sayfa numarası
    100   // sayfa boyutu
);

logs.forEach(function(r) {
    ins.consoleLog(r.getDttm() + " → " + r.getValue());
});
```

### `ins.getLoggedVariableValuesByPageAsc(names[], startDate, endDate, page, pageSize)`

Aynı arama **eskiden yeniye** (ASC) sıralı.

### `LoggedVariableValueDto` Alanları

| Alan / Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` / `.name` | `String` | Değişken adı |
| `getValue()` / `.value` | `Double` | Sayısal değer |
| `getTextValue()` / `.textValue` | `String` | Metinsel değer (değişken tipi string ise) |
| `getDttm()` / `.dttm` | `Date` | Kaydın zaman damgası |
| `getTime()` / `.time` | `Long` | Epoch ms |
| `getVariableId()` / `.variableId` | `String` | Değişken ID |
| `getProject()` / `.project` | `String` | Proje adı |
| `getProjectId()` / `.projectId` | `String` | Proje ID |

### `ins.getLoggedVariableNames()` / `(projectName)`

Loglanmış değişken adlarının listesi — ne'nin loglandığını keşfetmek için.

```javascript
var loggedOnes = ins.getLoggedVariableNames();
ins.consoleLog(loggedOnes.size() + " değişken loglanıyor");
```

## Tarihsel İstatistikler

Hepsi aynı imzayı paylaşır: `(variableNames[], startDate, endDate)` + opsiyonel `projectName` öntaki.

### `ins.getLoggedVariableValueStats(names[], startDate, endDate)`

Verilen aralık için **tek** istatistik seti — `Map<String, LoggedVariableValueStatsDto>`.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 3600000);   // son 1 saat

var stats = ins.getLoggedVariableValueStats(["ActivePower_kW"], start, end);
var s = stats.ActivePower_kW;
ins.consoleLog("avg=" + s.getAvgValue() + " min=" + s.getMinValue() + " max=" + s.getMaxValue());
```

### `ins.getLoggedHourlyVariableValueStats(names[], startDate, endDate)`

**Saatlik** gruplama — `Map<String, List<LoggedVariableValueStatsDto>>`. Her değişken için saat saat bir liste.

```javascript
var hourly = ins.getLoggedHourlyVariableValueStats(["ActivePower_kW"], start, end);
hourly.ActivePower_kW.forEach(function(s) {
    ins.consoleLog(s.getDttm() + " avg=" + s.getAvgValue());
});
```

### `ins.getLoggedDailyVariableValueStats(names[], startDate, endDate)`

**Günlük** gruplama — aynı yapı.

### `ins.getLoggedVariableValueStatsByInterval(names[], startDate, endDate, interval)`

Keyfi aralıkta gruplama — `interval` milisaniye cinsinden. Dönüş `Collection<LoggedVariableValueStatsDto>`.

```javascript
// 5 dakikalık pencerelerde istatistik
var bucket5min = ins.getLoggedVariableValueStatsByInterval(
    ["ActivePower_kW"], start, end, 5 * 60 * 1000
);
```

### `LoggedVariableValueStatsDto` Alanları

| Alan / Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` / `.name` | `String` | Değişken adı |
| `getVariableId()` / `.variableId` | `String` | Değişken ID |
| `getDttm()` / `.dttm` | `Date` | Bucket başlangıç zamanı |
| `getMinValue()` | `Double` | Minimum |
| `getMaxValue()` | `Double` | Maksimum |
| `getAvgValue()` | `Double` | Ortalama |
| `getSumValue()` | `Double` | Toplam |
| `getCountValue()` | `Double` | Kayıt sayısı |
| `getMedianValue()` | `Double` | Medyan |
| `getMiddleValue()` | `Double` | Orta (min+max)/2 |
| `getFirstValue()` | `Double` | İlk değer |
| `getLastValue()` | `Double` | Son değer |
| `getMaxDiffValue()` | `Double` | Max − Min |
| `getLastFirstDiffValue()` | `Double` | Son − İlk |

## Değişken Metadata

### `ins.getVariables()` / `(projectName)`

Projedeki tüm değişken tanımlarını döner — `Collection<VariableResponseDto>`.

```javascript
var list = ins.getVariables();
ins.consoleLog("Toplam " + list.size() + " değişken");
```

### `ins.getVariablesByConnectionName(connectionName)`

Belirli bir bağlantıya ait değişkenler.

### `ins.getVariablesByDeviceName(connectionName, deviceName)`

Belirli bir cihaza ait değişkenler.

### `ins.getVariablesByFrameName(connectionName, deviceName, frameName)`

Belirli bir frame'e ait değişkenler.

```javascript
var pLoc = ins.getVariablesByFrameName("MODBUS-PLC", "Device1", "HoldingRegs_0_100");
pLoc.forEach(function(v) {
    ins.consoleLog(v.getName() + " — " + v.getDsc());
});
```

### `ins.getVariable(name)`

Tek değişkenin tanımı — `VariableResponseDto`.

```javascript
var v = ins.getVariable("ActivePower_kW");
ins.consoleLog(v.getName() + " — " + v.getDsc());
```

### `ins.updateVariable(name, dto)`

Değişkenin konfigürasyonunu günceller — **tam DTO** gerektirir. Tipik kalıp: `getVariable` ile oku, alanı değiştir, `updateVariable` ile geri yaz.

```javascript
var v = ins.getVariable("ActivePower_kW");
v.setDsc("Total active power — updated");
// Diğer setter'lar v.setActiveFlag(true), v.setLogType(...) vb.
ins.updateVariable("ActivePower_kW", v);
```

## Örnek: Son 1 Saatlik Güç Ortalamasını Yaz

```javascript
function main() {
    var end = ins.now();
    var start = ins.getDate(end.getTime() - 3600000);

    var stats = ins.getLoggedVariableValueStats(["ActivePower_kW"], start, end);
    var avg = stats.ActivePower_kW.getAvgValue();

    ins.setVariableValue("ActivePower_1h_Avg", { value: avg });
    ins.writeLog("INFO", "PowerStats", "1h avg = " + avg.toFixed(2) + " kW");
}
main();
```
