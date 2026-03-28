---
title: "Alarm API"
description: "Aktif alarmlar, alarm geçmişi, onaylama ve zorla kapatma endpoint'leri"
sidebar:
  order: 6
---

Alarm API, aktif alarm sorgulama, alarm geçmişi ve alarm yönetimi sağlar.

## Aktif Alarmlar

### GET /api/alarms/fired-alarms

Aktif (tetiklenmiş) alarmları sayfalı olarak listeler.

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **projectId** | Integer | Proje ID'si |
| **page** | Integer | Sayfa numarası (varsayılan: 0) |
| **size** | Integer | Sayfa boyutu (varsayılan: 20) |

```bash
curl -b cookies.txt \
  "http://localhost:8081/api/alarms/fired-alarms?projectId=153" \
  -H "X-Space: claude"
```

Yanıt:
```json
{
  "content": [],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20
  },
  "totalPages": 0,
  "totalElements": 0,
  "empty": true
}
```

:::note
Aktif alarm olmadığında `content` boş dizi döner.
:::

### GET /api/alarms/fired-alarms/all

Tüm alarm geçmişini listeler (kapanmış alarmlar dahil).

## Alarm Yönetimi

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| POST | `/api/alarms/fired-alarms/acknowledge` | Alarm onayla (acknowledge) |
| POST | `/api/alarms/fired-alarms/force-off` | Alarm zorla kapat |

### Alarm Onaylama

```bash
curl -b cookies.txt -X POST \
  http://localhost:8081/api/alarms/fired-alarms/acknowledge \
  -H "X-Space: claude" -H "Content-Type: application/json" \
  -d '{"id": 12345}'
```

## Alarm Tanımı CRUD

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/alarms?projectId=X` | Alarm tanım listesi |
| POST | `/api/alarms` | Yeni alarm tanımla |
| PUT | `/api/alarms/{id}` | Alarm güncelle |
| DELETE | `/api/alarms/{id}` | Alarm sil |

## Alarm Grupları

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/alarm-groups?projectId=X` | Alarm grubu listesi |
| POST | `/api/alarm-groups` | Yeni grup oluştur |
| PUT | `/api/alarm-groups/{id}` | Grup güncelle |
| DELETE | `/api/alarm-groups/{id}` | Grup sil |
| POST | `/api/alarm-groups/{id}/activate` | Grubu etkinleştir |
| POST | `/api/alarm-groups/{id}/deactivate` | Grubu devre dışı bırak |
