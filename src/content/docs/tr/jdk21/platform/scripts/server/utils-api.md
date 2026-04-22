---
title: "Utils API"
description: "uuid, tarih, bit işlemleri ve sayı/metin formatlama"
sidebar:
  order: 17
---

Utils API küçük yardımcı fonksiyonları barındırır: UUID üretme, tarih, bit işlemleri, sayı formatlama, metin pad.

:::note
JDK11'de Utils altındaydı ama JDK21'de ayrı alt API'lere taşındı: **REST / ping / dosya / JSON** → [I/O Utils API](/docs/tr/jdk21/platform/scripts/server/io-utils-api/), **SQL / InfluxQL** → [Datasource API](/docs/tr/jdk21/platform/scripts/server/datasource-api/), **numpad / confirm / popup / vb.** (UI) → yalnızca tarayıcıda — [Client UI API](/docs/tr/jdk21/platform/scripts/client/ui-api/).
:::

## Benzersiz Kimlik

### `ins.uuid()`

Yeni bir UUID string'i üretir.

```javascript
var id = ins.uuid();
// → "f4cbb047-4376-4d8b-ae46-5cafed31155b"
```

## Tarih

### `ins.now()`

Sunucunun güncel zamanını `Date` olarak döndürür.

```javascript
var n = ins.now();
// → Wed Apr 23 00:15:42 TRT 2026
```

### `ins.getDate(ms)`

Verilen epoch milisaniyesine karşılık gelen `Date`'i döndürür.

```javascript
var yesterday = ins.getDate(Date.now() - 86400000);
var oneHourAgo = ins.getDate(Date.now() - 3600000);
```

## Bit İşlemleri

### `ins.getBit(value, bitIndex)`

Bir `Long` değerin belirli bit'ini (0 = en sağdaki) okur — `Boolean` döner.

```javascript
var statusWord = ins.getVariableValue("status_register").value;
var bit3 = ins.getBit(statusWord, 3);   // true | false
```

### `ins.setBit(value, bitIndex, bitValue)`

Bir `Long` değerin belirli bit'ini değiştirir — yeni `Long` döner (orijinal değer değişmez).

```javascript
var word = 0;
word = ins.setBit(word, 0, true);   // bit 0 = 1 → 1
word = ins.setBit(word, 3, true);   // bit 3 = 1 → 9
```

## Metin ve Sayı Formatlama

### `ins.leftPad(str, len, padChar)`

Metnin soluna `padChar` ekleyerek `len` uzunluğa tamamlar.

```javascript
ins.leftPad("42", 5, "0");    // → "00042"
ins.leftPad("AB", 4, " ");    // → "  AB"
```

### `ins.formatNumber(number, pattern, decimalSeparator, groupingSeparator)`

Sayıyı Java `DecimalFormat` pattern'i ile formatlar.

```javascript
ins.formatNumber(1234567.89, "#,##0.00", ",", ".");
// → "1.234.567,89"

ins.formatNumber(3.14159, "0.00", ".", ",");
// → "3.14"
```
