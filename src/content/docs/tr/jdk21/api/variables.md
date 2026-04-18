---
title: "Variable API"
description: "Değişken anlık değer okuma/yazma, tarihsel veri sorgulama ve istatistik endpoint'leri"
sidebar:
  order: 5
---

Variable API, inSCADA değişkenlerinin anlık ve tarihsel değerlerine REST üzerinden erişim sağlar.

## Anlık Değer Okuma

### GET /api/variables/{id}/value

Tek bir değişkenin anlık değerini okur.

```bash
curl -b cookies.txt http://localhost:8081/api/variables/23227/value \
  -H "X-Space: claude"
```

Yanıt:
```json
{
  "flags": { "scaled": true },
  "date": "2026-03-28T11:50:25.959+03:00",
  "value": 330.48,
  "extras": { "raw_value": 606.56 },
  "variableShortInfo": {
    "dsc": "Total active power",
    "frame": "Energy-Frame",
    "project": "Energy Monitoring Demo",
    "device": "Energy-Device",
    "name": "ActivePower_kW",
    "connection": "LOCAL-Energy"
  },
  "dateInMs": 1774687825959
}
```

### GET /api/variables/values

Birden fazla değişkenin anlık değerini toplu okur.

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **projectId** | Integer | Proje ID'si |
| **names** | String | Virgülle ayrılmış değişken adları |

```bash
curl -b cookies.txt \
  "http://localhost:8081/api/variables/values?projectId=153&names=ActivePower_kW,Voltage_V,Current_A" \
  -H "X-Space: claude"
```

Yanıt:
```json
{
  "ActivePower_kW": {
    "flags": { "scaled": true },
    "date": "2026-03-28T11:50:33.950+03:00",
    "value": 323.66,
    "extras": { "raw_value": 606.56 },
    "variableShortInfo": {
      "name": "ActivePower_kW",
      "dsc": "Total active power",
      "project": "Energy Monitoring Demo"
    },
    "dateInMs": 1774687833950
  },
  "Voltage_V": {
    "flags": { "scaled": true },
    "value": 236.7,
    "extras": { "raw_value": 228.0 },
    "variableShortInfo": { "name": "Voltage_V", "dsc": "Line voltage" }
  },
  "Current_A": {
    "flags": { "scaled": true },
    "value": 33.12,
    "extras": { "raw_value": 58.92 },
    "variableShortInfo": { "name": "Current_A", "dsc": "Line current" }
  }
}
```

## Değer Yazma

### POST /api/variables/{id}/value

Bir değişkene değer yazar.

```bash
curl -b cookies.txt -X POST http://localhost:8081/api/variables/23234/value \
  -H "X-Space: claude" -H "Content-Type: application/json" \
  -d '{"value": 55.0}'
```

## Tarihsel Veri

### GET /api/variables/loggedValues

Loglanan değişken verilerini sayfalı olarak sorgular.

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **projectId** | Integer | Proje ID'si |
| **names** | String | Virgülle ayrılmış değişken adları |
| **startDate** | Long | Başlangıç zamanı (epoch ms) |
| **endDate** | Long | Bitiş zamanı (epoch ms) |
| **page** | Integer | Sayfa numarası (0'dan başlar) |
| **size** | Integer | Sayfa boyutu |

### GET /api/variables/loggedValues/stats

Belirli aralıktaki istatistikleri döndürür.

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **projectId** | Integer | Proje ID'si |
| **names** | String | Virgülle ayrılmış değişken adları |
| **startDate** | Long | Başlangıç zamanı (epoch ms) |
| **endDate** | Long | Bitiş zamanı (epoch ms) |

:::tip
İstatistik yanıtı `avgValue`, `minValue`, `maxValue`, `sumValue`, `countValue`, `medianValue`, `firstValue`, `lastValue` alanlarını içerir. Detaylı yapı için [Script API — Variable API](/tr/platform/scripts/variable-api/) sayfasına bakın.
:::

### GET /api/variables/loggedValues/stats/hourly

Saatlik gruplanmış istatistikler.

### GET /api/variables/loggedValues/stats/daily

Günlük gruplanmış istatistikler.

## Değişken CRUD

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/variables?projectId=X` | Değişken listesi |
| GET | `/api/variables/{id}` | Değişken detayı |
| POST | `/api/variables` | Yeni değişken oluştur |
| PUT | `/api/variables/{id}` | Değişken güncelle |
| DELETE | `/api/variables/{id}` | Değişken sil |

### Değişken Listesi Yanıtı

```bash
curl -b cookies.txt "http://localhost:8081/api/variables?projectId=153" \
  -H "X-Space: claude"
```

Yanıt (kısaltılmış):
```json
[
  {
    "id": 23227,
    "name": "ActivePower_kW",
    "dsc": "Total active power",
    "type": "Float",
    "unit": "kW",
    "projectId": 153,
    "connectionId": 153,
    "deviceId": 453,
    "frameId": 703,
    "logType": "Periodically",
    "logPeriod": 10,
    "engZeroScale": 0.0,
    "engFullScale": 1000.0,
    "fractionalDigitCount": 2,
    "isActive": true,
    "keepLastValues": true
  },
  {
    "id": 23229,
    "name": "Voltage_V",
    "dsc": "Line voltage",
    "type": "Float",
    "unit": "V",
    "engZeroScale": 200.0,
    "engFullScale": 260.0
  }
]
```
