---
title: "Get"
description: "Değişken değerini metin olarak gösterme"
sidebar:
  order: 10
---

**Get**, bir SVG text öğesinin içeriğini değişken değeriyle güncelleyen en temel animation tipidir. Sayısal gösterge, etiket, durum metni gibi tüm metin tabanlı gösterimler için kullanılır.

## Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Get |
| **Uygun SVG Öğeleri** | `<text>`, `<tspan>` |
| **Expression Type** | Tag, Expression, Switch, Text |

## SVG Hazırlığı

```xml
<text id="power_display" x="100" y="50"
      font-size="24" fill="#333" text-anchor="middle">--</text>
```

## Yapılandırma Örnekleri

### Basit — Tag ile Doğrudan Bağlama

Expression Type: **Tag**
```
ActivePower_kW
```
Sonuç: `359.91` (değişkenin ham değeri)

### Formatlı — Expression ile

Expression Type: **Expression**
```javascript
var val = ins.getVariableValue("ActivePower_kW");
return val.value.toFixed(1) + " kW";
```
Sonuç: `359.9 kW`

### Birim ve Ondalık

```javascript
var val = ins.getVariableValue("Temperature_C");
return val.value.toFixed(1) + " °C";
```
Sonuç: `45.2 °C`

### Koşullu Metin — Switch ile

Expression Type: **Switch**
```
0 → Durdu
1 → Çalışıyor
2 → Arıza
3 → Bakım
```

### Boolean Durum Metni

```javascript
var status = ins.getVariableValue("GridStatus").value;
return status ? "ONLINE" : "OFFLINE";
```

### Zaman Damgası

```javascript
var val = ins.getVariableValue("ActivePower_kW");
var d = new Date(val.dateInMs);
var h = ("0" + d.getHours()).slice(-2);
var m = ("0" + d.getMinutes()).slice(-2);
var s = ("0" + d.getSeconds()).slice(-2);
return h + ":" + m + ":" + s;
```
Sonuç: `14:32:05`

### Birden Fazla Değişken

```javascript
var p = ins.getVariableValue("ActivePower_kW").value;
var v = ins.getVariableValue("Voltage_V").value;
var i = ins.getVariableValue("Current_A").value;
return p.toFixed(0) + " kW | " + v.toFixed(0) + " V | " + i.toFixed(1) + " A";
```
Sonuç: `360 kW | 235 V | 36.2 A`

## GetSymbol

**GetSymbol** tipi, Space seviyesindeki Symbol kütüphanesinden SVG sembol yükler. Get'in metin yerine görsel sembol versiyonudur.

| Alan | Değer |
|------|-------|
| **Type** | GetSymbol |
| **Expression Type** | Expression |
| **Expression** | Sembol adı |

Kullanım: Cihaz tipine göre dinamik ikon gösterme.
