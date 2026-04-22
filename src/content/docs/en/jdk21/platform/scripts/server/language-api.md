---
title: "Language API"
description: "Fetch translated text from the platform's localization catalog"
sidebar:
  order: 14
---

Language API fetches a translation from the platform's localization catalog — useful when notifications / reports / log messages produced by a script should match the active user's language.

## `ins.loc(lang, key)`

Returns the translation for `key` in the given language code (`tr`, `en`, …). If no match is found, the key is returned verbatim.

```javascript
var title = ins.loc("tr", "alarm.active");   // → "Aktif alarm"
var titleEn = ins.loc("en", "alarm.active"); // → "Active alarm"
```

Example — notification in the session user's language:

```javascript
var lang = user.activeSpace && user.activeSpace.language
    ? user.activeSpace.language
    : "en";
var msg = ins.loc(lang, "shift.end.summary");
ins.sendMail([user.name], ins.loc(lang, "shift.end.title"), msg);
```
