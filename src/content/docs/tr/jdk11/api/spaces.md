---
title: "Space API"
description: "Space (çalışma alanı) CRUD endpoint'leri"
sidebar:
  order: 3
---

Space, inSCADA'nın çoklu çalışma alanı (multi-tenant) yapısıdır. Her space bağımsız projeler, kullanıcılar ve yapılandırmalar içerir.

## Space Listesi

### GET /api/spaces

Tüm space'leri listeler.

```bash
curl -b cookies.txt http://localhost:8081/api/spaces \
  -H "X-Space: default_space"
```

Yanıt:
```json
[
  {
    "id": 1,
    "name": "default_space",
    "createdBy": "inscada",
    "creationDate": "2024-04-24T11:05:55.076+03:00"
  },
  {
    "id": 2,
    "name": "claude",
    "createdBy": "inscada",
    "creationDate": "2026-03-04T13:36:31.143+03:00"
  }
]
```

## Space CRUD

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/spaces` | Space listesi |
| GET | `/api/spaces/{id}` | Space detayı |
| POST | `/api/spaces` | Yeni space oluştur |
| PUT | `/api/spaces/{id}` | Space güncelle |
| DELETE | `/api/spaces/{id}` | Space sil |

### Space Oluşturma

```bash
curl -b cookies.txt -X POST http://localhost:8081/api/spaces \
  -H "Content-Type: application/json" \
  -d '{"name": "production"}'
```

## X-Space Header

Tüm API isteklerinde hangi space'te çalışıldığını belirtmek için `X-Space` header'ı kullanılır:

```
X-Space: default_space
```

Bu header gönderilmezse varsayılan space kullanılır.
