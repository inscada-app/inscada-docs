---
title: "REST API Overview"
description: "inSCADA REST API — kimlik doğrulama, endpoint yapısı ve kullanım"
sidebar:
  order: 1
---

inSCADA, tamamen RESTful bir mimariye sahiptir. Platformdaki her işlem — değişken okuma/yazma, proje yönetimi, alarm sorgulama, bağlantı kontrolü — REST API üzerinden gerçekleştirilebilir.

## API Erişimi

**Base URL:**
```
https://<inscada-ip>:8082/api/
```

Tüm endpoint'ler `/api/` prefix'i ile başlar. API, JSON formatında veri alır ve döndürür.

## Kimlik Doğrulama

inSCADA REST API, form tabanlı login ve JWT token ile kimlik doğrulama kullanır.

### 1. Login

```
POST /login
Content-Type: multipart/form-data

username=admin&password=admin
```

Başarılı yanıt, `ins_access_token` ve `ins_refresh_token` cookie'lerini set eder.

### 2. API İstekleri

Login sonrası alınan cookie'ler sonraki isteklerde otomatik gönderilir. Alternatif olarak token header ile de gönderilebilir:

```
GET /api/projects
Cookie: ins_access_token=<token>; ins_refresh_token=<token>
X-Space: default_space
```

### 3. Space Header

Çoklu çalışma alanı (multi-tenant) yapısında, hangi space'te çalışıldığını belirtmek için `X-Space` header'ı kullanılır:

```
X-Space: default_space
```

## Endpoint Kategorileri

inSCADA REST API, 91 controller ve 1100+ endpoint içerir. Aşağıda ana kategoriler ve temel endpoint'ler listelenmiştir:

### Kimlik Doğrulama ve Kullanıcı

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| POST | `/login` | Kullanıcı girişi |
| GET | `/api/auth/currentUser` | Oturum açmış kullanıcı bilgisi |
| GET | `/api/auth/loggedInUsers` | Aktif oturum listesi |
| GET | `/api/users` | Kullanıcı listesi |
| POST | `/api/users` | Yeni kullanıcı oluştur |
| PUT | `/api/users/{id}` | Kullanıcı güncelle |
| DELETE | `/api/users/{id}` | Kullanıcı sil |
| GET | `/api/users/{id}/roles` | Kullanıcının rolleri |

### Space (Çalışma Alanı)

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/spaces` | Space listesi |
| GET | `/api/spaces/{id}` | Space detayı |
| POST | `/api/spaces` | Yeni space oluştur |
| PUT | `/api/spaces/{id}` | Space güncelle |
| DELETE | `/api/spaces/{id}` | Space sil |

### Proje

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/projects` | Proje listesi |
| GET | `/api/projects/{id}` | Proje detayı |
| GET | `/api/projects/{id}/status` | Proje durumu |
| POST | `/api/projects` | Yeni proje oluştur |
| PUT | `/api/projects/{id}` | Proje güncelle |
| DELETE | `/api/projects/{id}` | Proje sil |
| POST | `/api/projects/clone` | Proje klonla |

### Bağlantı (Connection)

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| POST | `/api/connections/{id}/start` | Bağlantıyı başlat |
| POST | `/api/connections/{id}/stop` | Bağlantıyı durdur |
| GET | `/api/connections/{id}/status` | Bağlantı durumu |
| GET | `/api/connections/{id}/browse` | Cihaz keşfi (OPC UA, OPC DA) |

Her protokolün kendi CRUD endpoint'leri vardır:
- `/api/modbus/connections`, `/api/dnp3/connections`, `/api/iec104/connections` ...
- `/api/modbus/variables`, `/api/dnp3/variables` ...

### Değişken Değerleri

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/variables/{id}/value` | Tek değişken anlık değeri |
| GET | `/api/variables/values?projectId=X&names=a,b` | Toplu anlık değer |
| POST | `/api/variables/{id}/value` | Değişkene değer yaz |
| GET | `/api/variables/loggedValues` | Tarihsel veri sorgula |
| GET | `/api/variables/loggedValues/stats` | İstatistik (avg, min, max) |
| GET | `/api/variables/loggedValues/stats/hourly` | Saatlik istatistik |

### Alarm

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/alarms/fired-alarms` | Aktif alarmlar |
| GET | `/api/alarms/fired-alarms/all` | Tüm alarm geçmişi |
| POST | `/api/alarms/fired-alarms/acknowledge` | Alarm onayla |
| POST | `/api/alarms/fired-alarms/force-off` | Alarm zorla kapat |

### Script

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/scripts` | Script listesi |
| GET | `/api/scripts/{id}` | Script detayı |
| POST | `/api/scripts` | Yeni script oluştur |
| PUT | `/api/scripts/{id}` | Script güncelle |
| DELETE | `/api/scripts/{id}` | Script sil |
| GET | `/api/scripts/{id}/status` | Script çalışma durumu |
| POST | `/api/scripts/runner` | Script kodu çalıştır (ad-hoc) |

### Animasyon (SVG Ekranlar)

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/animations` | Animasyon listesi |
| GET | `/api/animations/{id}` | Animasyon detayı |
| POST | `/api/animations` | Yeni animasyon oluştur |
| PUT | `/api/animations/{id}` | Animasyon güncelle |
| PUT | `/api/animations/{id}/svg` | SVG içeriği güncelle |
| POST | `/api/animations/{id}/clone` | Animasyon klonla |

### Trend

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/trends` | Trend listesi |
| POST | `/api/trends` | Yeni trend oluştur |
| GET | `/api/trends/{id}/tags` | Trend tag'leri |
| POST | `/api/trends/{id}/tags` | Tag ekle |

### Custom Menu

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/custom-menus` | Menü listesi |
| POST | `/api/custom-menus` | Yeni menü oluştur |
| PUT | `/api/custom-menus/{id}` | Menü güncelle |
| DELETE | `/api/custom-menus/{id}` | Menü sil |

### Rapor

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/reports` | Rapor listesi |
| POST | `/api/reports/{id}/run` | Rapor çalıştır |
| GET | `/api/reports/{id}/export/pdf` | PDF dışa aktar |
| GET | `/api/reports/{id}/export/excel` | Excel dışa aktar |

### Sistem

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/api/version` | Platform versiyonu |
| GET | `/api/license` | Lisans bilgisi |
| GET | `/api/cluster/leader` | Cluster lider node |
| GET | `/api/system/status` | Sistem durumu |

### Örnek: Versiyon Sorgulama

```bash
curl -b cookies.txt http://localhost:8081/api/version -H "X-Space: default_space"
```

```
20260311-1-jdk11
```

## Swagger / API Dokümantasyonu

inSCADA, yerleşik **Swagger UI** ile interaktif API dokümantasyonu sağlar. Geliştirme modunda (`dev` profili) aktifleşir:

```
https://<inscada-ip>:8082/swagger-ui/
```

Swagger UI üzerinden tüm endpoint'leri keşfedebilir, parametre açıklamalarını görebilir ve doğrudan test istekleri gönderebilirsiniz.

## Yanıt Formatı

Tüm yanıtlar JSON formatındadır:

```json
// Başarılı yanıt
{
    "id": 1,
    "name": "Temperature",
    "value": 25.4,
    ...
}

// Hata yanıtı
{
    "status": 400,
    "error": "Bad Request",
    "message": "Variable not found: invalid_name"
}
```

### HTTP Durum Kodları

| Kod | Açıklama |
|-----|----------|
| **200** | Başarılı |
| **201** | Oluşturuldu |
| **400** | Hatalı istek |
| **401** | Kimlik doğrulama gerekli |
| **403** | Yetkisiz erişim |
| **404** | Bulunamadı |
| **500** | Sunucu hatası |

## Rate Limiting

API istekleri rate-limit ile korunur. Aşırı istek durumunda `429 Too Many Requests` yanıtı döner. Limit değerleri sistem yapılandırmasından ayarlanabilir.
