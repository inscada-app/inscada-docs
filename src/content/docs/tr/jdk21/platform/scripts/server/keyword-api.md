---
title: "Keyword API"
description: "Scriptten meta veri (keyword) kaydı"
sidebar:
  order: 15
---

Keyword API, script çalışırken serbest biçimli meta veri (keyword) kaydı yapar. Tipik kullanım: batch işleminin, vardiyanın, kalibrasyon döngüsünün sonucunu anahtar-değer olarak saklamak ve raporda/arama'da geri çağırmak.

## `ins.saveKeyword(objectMap)`

Anahtar-değer haritasını platform keyword deposuna yazar (`Map<String, Object>`).

```javascript
ins.saveKeyword({
    "batch_id": "B-240423-A",
    "operator": user.name,
    "end_temperature": 85.4,
    "duration_sec": 1245,
    "status": "ok"
});
```

Harita içindeki değerler hem ilkel (sayı, string, boolean) hem de nested obje/dizi olabilir.
