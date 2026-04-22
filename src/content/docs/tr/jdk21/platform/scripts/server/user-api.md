---
title: "User API"
description: "Kullanıcı listesi, aktif oturumlar ve kimlik doğrulama denemeleri"
sidebar:
  order: 10
---

User API, platformdaki kullanıcıları ve oturum durumunu sorgular. Aktif kullanıcı **kendisinin** bilgisine `ins.*` ile değil, script context'indeki `user` global'i üzerinden erişir.

## `ins.getAllUsernames()`

Sistemdeki tüm kullanıcı adlarının koleksiyonunu döner (`Collection<String>`).

```javascript
var users = ins.getAllUsernames();
users.forEach(function(u) {
    ins.consoleLog(u);
});
```

## `ins.getLoggedInUsers()`

Şu anda oturum açmış olan kullanıcıların koleksiyonunu döner.

```javascript
var online = ins.getLoggedInUsers();
ins.consoleLog("Aktif kullanıcı sayısı: " + online.size());
```

## `ins.getLastAuthAttempts()`

Son kimlik doğrulama denemelerini (`AuthAttemptDto`) döner — başarılı ve başarısız dahil.

```javascript
var attempts = ins.getLastAuthAttempts();
attempts.forEach(function(a) {
    ins.consoleLog(a.username + " @ " + a.remoteAddress + " → " + (a.success ? "OK" : "FAIL"));
});
```

`AuthAttemptDto` tipik alanları: `username`, `remoteAddress`, `success`, `createdAt`.
