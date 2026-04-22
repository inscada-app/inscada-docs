---
title: "Utils API"
description: "uuid, date, bit operations, and number/string formatting"
sidebar:
  order: 17
---

Utils API contains small utility helpers: UUID generation, date, bit operations, number formatting, string padding.

:::note
In JDK11 these lived under Utils, but JDK21 split them into dedicated sub-APIs: **REST / ping / file / JSON** → [I/O Utils API](/docs/en/jdk21/platform/scripts/server/io-utils-api/), **SQL / InfluxQL** → [Datasource API](/docs/en/jdk21/platform/scripts/server/datasource-api/), **numpad / confirm / popup / etc.** (UI) → browser-only — [Client UI API](/docs/en/jdk21/platform/scripts/client/ui-api/).
:::

## Unique Identifier

### `ins.uuid()`

Generates a fresh UUID string.

```javascript
var id = ins.uuid();
// → "f4cbb047-4376-4d8b-ae46-5cafed31155b"
```

## Date

### `ins.now()`

Returns the server's current time as a `Date`.

```javascript
var n = ins.now();
// → Wed Apr 23 00:15:42 TRT 2026
```

### `ins.getDate(ms)`

Returns the `Date` that corresponds to the given epoch millisecond.

```javascript
var yesterday = ins.getDate(Date.now() - 86400000);
var oneHourAgo = ins.getDate(Date.now() - 3600000);
```

## Bit Operations

### `ins.getBit(value, bitIndex)`

Reads a specific bit (0 = rightmost) from a `Long` — returns `Boolean`.

```javascript
var statusWord = ins.getVariableValue("status_register").value;
var bit3 = ins.getBit(statusWord, 3);   // true | false
```

### `ins.setBit(value, bitIndex, bitValue)`

Flips a specific bit of a `Long` — returns the new `Long` (the original is unchanged).

```javascript
var word = 0;
word = ins.setBit(word, 0, true);   // bit 0 = 1 → 1
word = ins.setBit(word, 3, true);   // bit 3 = 1 → 9
```

## String and Number Formatting

### `ins.leftPad(str, len, padChar)`

Pads the string on the left with `padChar` to reach `len`.

```javascript
ins.leftPad("42", 5, "0");    // → "00042"
ins.leftPad("AB", 4, " ");    // → "  AB"
```

### `ins.formatNumber(number, pattern, decimalSeparator, groupingSeparator)`

Formats a number using a Java `DecimalFormat` pattern.

```javascript
ins.formatNumber(1234567.89, "#,##0.00", ",", ".");
// → "1.234.567,89"

ins.formatNumber(3.14159, "0.00", ".", ",");
// → "3.14"
```
