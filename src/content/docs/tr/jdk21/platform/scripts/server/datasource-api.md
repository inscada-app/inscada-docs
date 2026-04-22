---
title: "Datasource API"
description: "Server-side SQL ve InfluxQL sorguları — relational ve zaman serisi veri kaynakları"
sidebar:
  order: 11
---

Datasource API, script'ler içinden relational (SQL) ve time-series (InfluxQL) veri kaynaklarına sorgu gönderir. Tek bir çağrı **birden fazla result set** dönebilir — her sorgu sonucu bir `ResultDto` olarak sarılır.

## SQL Sorguları

### `ins.runSql(sql)`

Varsayılan veritabanında SQL sorgusu çalıştırır.

```javascript
var qr = ins.runSql("SELECT id, name, value FROM custom_table");
var rows = qr.getResult(0).getRows();
// rows = [ {id: 1, name: "temp", value: 25.4}, ... ]
rows.forEach(function(row) {
    ins.consoleLog(row.id + ": " + row.name + " = " + row.value);
});
```

### `ins.runSql(datasourceName, sql)`

Tanımlı bir dış veri kaynağında çalıştırır.

```javascript
var qr = ins.runSql("erp_database", "SELECT production_count FROM daily_report");
```

### `ins.runSql(url, username, password, sql)`

Doğrudan JDBC bağlantı bilgisiyle çalıştırır.

```javascript
var qr = ins.runSql(
    "jdbc:postgresql://192.168.1.50:5432/erp",
    "reader", "password123",
    "SELECT * FROM orders WHERE status = 'active'"
);
```

## InfluxQL Sorguları

### `ins.runInfluxQL(influxQL)`

Varsayılan InfluxDB veri kaynağında sorgu çalıştırır.

```javascript
var qr = ins.runInfluxQL(
    "SELECT mean(value) FROM variable_values WHERE time > now() - 1h GROUP BY time(5m)"
);
qr.getResults().forEach(function(series) {
    ins.consoleLog(series.getName() + ": " + series.getValues().length + " points");
});
```

### `ins.runInfluxQL(datasourceName, influxQL)`

Tanımlı bir dış InfluxDB veri kaynağında çalıştırır.

### `ins.runInfluxQL(url, username, password, influxQL)`

Doğrudan InfluxDB bağlantı bilgisiyle çalıştırır.

## Kayıtlı Sorgu Şablonları

Platformda önceden tanımlanmış sorgu şablonlarını getirir — sorguyu çalıştırmaz, yalnızca metni döner.

```javascript
var sql = ins.getCustomSqlQuery("daily_report");
var qr = ins.runSql(sql);

var influxQL = ins.getCustomInfluxQLQuery("hourly_avg");
var qr2 = ins.runInfluxQL(influxQL);
```

## Sonuç Yapısı

`runSql` ve `runInfluxQL` her zaman bir `QueryResultDto` döner; içinde bir veya birden fazla `ResultDto` olur.

### `QueryResultDto`

| Metod | Dönüş | Açıklama |
| --- | --- | --- |
| `getSize()` | `Integer` | Result set sayısı |
| `getResult(index)` | `ResultDto` | Index'e göre tek bir result |
| `getResults()` | `List<ResultDto>` | Tüm result'lar |

### `ResultDto` — üç alt tür

Her result'ın `getType()` metodu tipini söyler: `"list"`, `"scalar"` veya `"series"`.

**1. `ListResultDto`** (`type = "list"`) — SELECT sorguları.

| Metod | Dönüş |
| --- | --- |
| `getRows()` | `List<Map<String, Object>>` — her satır kolon adı → değer |

```javascript
var r = ins.runSql("SELECT id, name FROM t").getResult(0);
// r.getType()  → "list"
var rows = r.getRows();
// rows[0] = { id: 1, name: "x" }
```

**2. `ScalarResultDto`** (`type = "scalar"`) — INSERT / UPDATE / DELETE ve benzeri DML.

| Metod | Dönüş |
| --- | --- |
| `getResult()` | `int` — etkilenen satır sayısı |

```javascript
var r = ins.runSql("UPDATE t SET x=1 WHERE id=2").getResult(0);
// r.getType()    → "scalar"
// r.getResult()  → 1
```

**3. `SeriesResultDto`** (`type = "series"`) — InfluxQL sonuçları.

| Metod | Dönüş |
| --- | --- |
| `getName()` | `String` — series adı (genelde measurement) |
| `getTags()` | `Map<String, String>` |
| `getColumns()` | `List<String>` — sütun adları |
| `getValues()` | `List<List<Object>>` — her satır kolon sırasında değerler |

```javascript
var series = ins.runInfluxQL("SELECT mean(value) FROM cpu GROUP BY host").getResult(0);
// series.getType()    → "series"
var cols = series.getColumns();   // ["time", "mean"]
series.getValues().forEach(function(row) {
    ins.consoleLog(row[0] + " → " + row[1]);
});
```
