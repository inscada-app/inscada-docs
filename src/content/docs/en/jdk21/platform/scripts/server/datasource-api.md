---
title: "Datasource API"
description: "Server-side SQL and InfluxQL queries — relational and time-series data sources"
sidebar:
  order: 11
---

Datasource API lets scripts query relational (SQL) and time-series (InfluxQL) data sources. A single call may return **multiple result sets** — each query result is wrapped in a `ResultDto`.

## SQL Queries

### `ins.runSql(sql)`

Runs a SQL query against the default database.

```javascript
var qr = ins.runSql("SELECT id, name, value FROM custom_table");
var rows = qr.getResult(0).getRows();
// rows = [ {id: 1, name: "temp", value: 25.4}, ... ]
rows.forEach(function(row) {
    ins.consoleLog(row.id + ": " + row.name + " = " + row.value);
});
```

### `ins.runSql(datasourceName, sql)`

Runs against a named external datasource.

```javascript
var qr = ins.runSql("erp_database", "SELECT production_count FROM daily_report");
```

### `ins.runSql(url, username, password, sql)`

Runs with an explicit JDBC connection.

```javascript
var qr = ins.runSql(
    "jdbc:postgresql://192.168.1.50:5432/erp",
    "reader", "password123",
    "SELECT * FROM orders WHERE status = 'active'"
);
```

## InfluxQL Queries

### `ins.runInfluxQL(influxQL)`

Runs an InfluxQL query against the default InfluxDB datasource.

```javascript
var qr = ins.runInfluxQL(
    "SELECT mean(value) FROM variable_values WHERE time > now() - 1h GROUP BY time(5m)"
);
qr.getResults().forEach(function(series) {
    ins.consoleLog(series.getName() + ": " + series.getValues().length + " points");
});
```

### `ins.runInfluxQL(datasourceName, influxQL)`

Runs against a named external InfluxDB datasource.

### `ins.runInfluxQL(url, username, password, influxQL)`

Runs with explicit InfluxDB connection credentials.

## Saved Query Templates

Fetches query templates defined on the platform — does not execute the query, only returns the text.

```javascript
var sql = ins.getCustomSqlQuery("daily_report");
var qr = ins.runSql(sql);

var influxQL = ins.getCustomInfluxQLQuery("hourly_avg");
var qr2 = ins.runInfluxQL(influxQL);
```

## Result Shape

`runSql` and `runInfluxQL` always return a `QueryResultDto` that carries one or more `ResultDto` instances.

### `QueryResultDto`

| Method | Returns | Description |
| --- | --- | --- |
| `getSize()` | `Integer` | Number of result sets |
| `getResult(index)` | `ResultDto` | A single result by index |
| `getResults()` | `List<ResultDto>` | All results |

### `ResultDto` — three concrete subtypes

Each result's `getType()` returns `"list"`, `"scalar"` or `"series"`.

**1. `ListResultDto`** (`type = "list"`) — SELECT queries.

| Method | Returns |
| --- | --- |
| `getRows()` | `List<Map<String, Object>>` — each row maps column name → value |

```javascript
var r = ins.runSql("SELECT id, name FROM t").getResult(0);
// r.getType()  → "list"
var rows = r.getRows();
// rows[0] = { id: 1, name: "x" }
```

**2. `ScalarResultDto`** (`type = "scalar"`) — INSERT / UPDATE / DELETE and other DML.

| Method | Returns |
| --- | --- |
| `getResult()` | `int` — affected row count |

```javascript
var r = ins.runSql("UPDATE t SET x=1 WHERE id=2").getResult(0);
// r.getType()    → "scalar"
// r.getResult()  → 1
```

**3. `SeriesResultDto`** (`type = "series"`) — InfluxQL results.

| Method | Returns |
| --- | --- |
| `getName()` | `String` — series name (usually the measurement) |
| `getTags()` | `Map<String, String>` |
| `getColumns()` | `List<String>` — column names |
| `getValues()` | `List<List<Object>>` — each row carries values in column order |

```javascript
var series = ins.runInfluxQL("SELECT mean(value) FROM cpu GROUP BY host").getResult(0);
// series.getType()    → "series"
var cols = series.getColumns();   // ["time", "mean"]
series.getValues().forEach(function(row) {
    ins.consoleLog(row[0] + " → " + row[1]);
});
```
