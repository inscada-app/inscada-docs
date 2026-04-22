---
title: "Language API"
description: "Lokalizasyon anahtarlarından çevrilmiş metin elde etme"
sidebar:
  order: 14
---

Language API, platformun lokalizasyon sözlüğünden bir anahtar üzerinden çevrilmiş metin çekmek için kullanılır — script'in ürettiği bildirim / rapor / log metinlerinin aktif kullanıcının diline göre üretilmesini sağlar.

## `ins.loc(lang, key)`

Belirtilen dil kodunda (`tr`, `en`, …) anahtara karşılık gelen metni döner. Eşleşme yoksa anahtar olduğu gibi döner.

```javascript
var title = ins.loc("tr", "alarm.active");   // → "Aktif alarm"
var titleEn = ins.loc("en", "alarm.active"); // → "Active alarm"
```

Örnek — oturumdaki kullanıcının diline göre bildirim:

```javascript
var lang = user.activeSpace && user.activeSpace.language
    ? user.activeSpace.language
    : "tr";
var msg = ins.loc(lang, "shift.end.summary");
ins.sendMail([user.name], ins.loc(lang, "shift.end.title"), msg);
```
