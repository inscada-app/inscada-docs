---
title: "Project API"
description: "Proje CRUD, durum sorgulama ve bağlantı yönetimi endpoint'leri"
sidebar:
  order: 4
---

## Proje Listesi

### GET /api/projects

Space'teki tüm projeleri listeler.

```bash
curl -b cookies.txt http://localhost:8081/api/projects \
  -H "X-Space: claude"
```

Yanıt:
```json
[
  {
    "id": 153,
    "name": "Energy Monitoring Demo",
    "dsc": "Demo project for energy monitoring with simulated data",
    "isActive": true,
    "latitude": null,
    "longitude": null,
    "createdBy": "inscada",
    "creationDate": "2026-03-04T14:47:52.913+03:00",
    "lastModifiedDate": "2026-03-04T14:47:52.913+03:00"
  },
  {
    "id": 353,
    "name": "Building",
    "dsc": "Building Management System - HVAC & Air Conditioning Demo",
    "isActive": true
  }
]
```

### GET /api/projects/{id}

Proje detayını getirir.

```bash
curl -b cookies.txt http://localhost:8081/api/projects/153 \
  -H "X-Space: claude"
```

Yanıt:
```json
{
  "id": 153,
  "name": "Energy Monitoring Demo",
  "dsc": "Demo project for energy monitoring with simulated data",
  "isActive": true,
  "icon": null,
  "address": null,
  "latitude": null,
  "longitude": null,
  "properties": null,
  "createdBy": "inscada",
  "creationDate": "2026-03-04T14:47:52.913+03:00"
}
```

### GET /api/projects/{id}/status

Projenin tüm bileşenlerinin çalışma durumunu döndürür.

```bash
curl -b cookies.txt http://localhost:8081/api/projects/153/status \
  -H "X-Space: claude"
```

Yanıt:
```json
{
  "connectionStatuses": { "153": "Connected" },
  "scriptStatuses": { "160": "Not Scheduled", "159": "Not Scheduled" },
  "dataTransferStatuses": {},
  "reportStatuses": {},
  "alarmGroupStatuses": {}
}
```

## Proje CRUD

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| POST | `/api/projects` | Yeni proje oluştur |
| PUT | `/api/projects/{id}` | Proje güncelle |
| DELETE | `/api/projects/{id}` | Proje sil |
| POST | `/api/projects/clone` | Proje klonla |

### Proje Oluşturma

```bash
curl -b cookies.txt -X POST http://localhost:8081/api/projects \
  -H "X-Space: claude" -H "Content-Type: application/json" \
  -d '{
    "name": "New Project",
    "dsc": "Project description",
    "isActive": true,
    "latitude": 41.0,
    "longitude": 29.0
  }'
```

## Bağlantılar

### GET /api/connections?projectId={id}

Projedeki bağlantıları listeler.

```bash
curl -b cookies.txt "http://localhost:8081/api/connections?projectId=153" \
  -H "X-Space: claude"
```

Yanıt:
```json
[
  {
    "id": 153,
    "name": "LOCAL-Energy",
    "protocol": "LOCAL",
    "ip": "127.0.0.1",
    "port": 0,
    "projectId": 153,
    "dsc": "Local protocol connection for energy simulation",
    "owner": "inscada"
  }
]
```

### Bağlantı Kontrolü

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| POST | `/api/connections/{id}/start` | Bağlantıyı başlat |
| POST | `/api/connections/{id}/stop` | Bağlantıyı durdur |
| GET | `/api/connections/{id}/status` | Bağlantı durumu |
