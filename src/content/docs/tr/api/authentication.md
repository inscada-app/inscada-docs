---
title: "Kimlik Doğrulama"
description: "inSCADA REST API kimlik doğrulama — login, token yönetimi ve güvenlik"
sidebar:
  order: 2
---

inSCADA REST API, form tabanlı login ve cookie tabanlı JWT token ile kimlik doğrulama kullanır.

## Login

### İstek

```
POST /login
Content-Type: multipart/form-data
```

| Alan | Değer |
|------|-------|
| **username** | Kullanıcı adı |
| **password** | Şifre |

### Yanıt

Başarılı login, yanıt header'ında iki cookie set eder:

| Cookie | Açıklama | Süre |
|--------|----------|------|
| **ins_access_token** | Erişim token'ı | Kısa (dakikalar) |
| **ins_refresh_token** | Yenileme token'ı | Uzun (saatler) |

```
HTTP/1.1 200 OK
Set-Cookie: ins_access_token=eyJhbG...; Path=/; HttpOnly
Set-Cookie: ins_refresh_token=eyJhbG...; Path=/; HttpOnly
```

### cURL Örneği

```bash
# Login
curl -c cookies.txt -X POST https://localhost:8082/login \
  -F "username=admin" -F "password=admin"

# API çağrısı (cookie ile)
curl -b cookies.txt https://localhost:8082/api/projects \
  -H "X-Space: default_space"
```

## Token Yenileme

Access token süresi dolduğunda, refresh token ile otomatik yenilenir. İstemci tarafında ek işlem gerekmez — tarayıcı cookie'leri otomatik gönderir.

Programatik erişimde cookie'leri saklayıp sonraki isteklerde gönderin:

```javascript
// JavaScript (fetch)
const response = await fetch('https://inscada:8082/api/projects', {
  credentials: 'include',
  headers: { 'X-Space': 'default_space' }
});
```

```python
# Python (requests)
import requests

session = requests.Session()
session.post('https://inscada:8082/login',
    data={'username': 'admin', 'password': 'admin'},
    verify=False)

projects = session.get('https://inscada:8082/api/projects',
    headers={'X-Space': 'default_space'}).json()
```

## X-Space Header

Çoklu çalışma alanı (multi-tenant) yapısında, API isteklerinde hangi space'te çalışıldığını belirtmek için `X-Space` header'ı kullanılır:

```
X-Space: default_space
```

Bu header her API isteğinde gönderilmelidir. Gönderilmezse varsayılan space kullanılır.

## Güvenlik

### Brute-Force Koruması

- **5 başarısız giriş** → hesap **10 dakika** kilitlenir
- Kilitlenen hesaplar `/api/auth/lockedUsers` endpoint'inden görüntülenebilir

### IP Filtreleme

Platform yöneticisi, whitelist/blacklist tabanlı IP filtreleme yapılandırabilir. Filtreleme aktifken yalnızca izin verilen IP adreslerinden API erişimi mümkündür.

### OTP / İki Faktörlü Doğrulama

Etkinleştirildiğinde, login sonrası OTP (One-Time Password) doğrulaması da gerekir. TOTP uyumlu mobil uygulamalar (Google Authenticator, Authy vb.) ile kullanılır.

### HTTPS

Tüm API trafiği **HTTPS** üzerinden şifrelenmelidir. HTTP (port 8081) yalnızca geliştirme ortamında kullanılmalıdır.
