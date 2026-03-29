---
title: "Color"
description: "Değere göre renk değiştirme"
sidebar:
  order: 11
---

**Color**, bir SVG öğesinin dolgu (fill) veya çizgi (stroke) rengini değişken değerine göre dinamik olarak değiştirir. Durum göstergesi, alarm renklendirme, eşik bazlı görselleştirme için kullanılır.

## Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Color |
| **Uygun SVG Öğeleri** | `<rect>`, `<circle>`, `<ellipse>`, `<polygon>`, `<path>`, `<text>` |
| **Expression Type** | Expression, Switch, Tetra Color |

## SVG Hazırlığı

```xml
<circle id="status_led" cx="50" cy="50" r="12" fill="#cccccc" stroke="#999"/>
```

## Yapılandırma Örnekleri

### Boolean Durum — Switch ile

Expression Type: **Switch**
```
true → #00cc00
false → #ff0000
```
Motor çalışıyorsa yeşil, durmuşsa kırmızı.

### Çoklu Durum — Switch ile

```
0 → #999999
1 → #00cc00
2 → #ff0000
3 → #ff8800
```
0=Kapalı (gri), 1=Çalışıyor (yeşil), 2=Arıza (kırmızı), 3=Uyarı (turuncu)

### Eşik Bazlı — Expression ile

Expression Type: **Expression**
```javascript
var temp = ins.getVariableValue("Temperature_C").value;
if (temp > 80) return "#ff0000";      // kırmızı — kritik
if (temp > 60) return "#ff8800";      // turuncu — uyarı
if (temp > 40) return "#ffcc00";      // sarı — dikkat
return "#00cc00";                     // yeşil — normal
```

### Gradient / Yanıp Sönme

İki renk arasında yanıp sönme efekti:
```javascript
return "#ff0000/#ffffff";  // kırmızı ↔ beyaz yanıp sönme
```

`/` karakteri ile iki renk belirtildiğinde SVG `<animate>` elementi oluşturulur ve renk geçişi yapılır.

### Tetra Color (Alarm 4 Renk)

Expression Type: **Tetra Color**

Alarm grubunun dört durumuna göre otomatik renklendirme:

| Durum | Varsayılan Renk |
|-------|----------------|
| Fired + No Ack | Kırmızı yanıp söner |
| Fired + Ack | Kırmızı sabit |
| Off + No Ack | Sarı |
| Off + Ack | Normal (gri/beyaz) |

Alarm grubu tanımındaki renk ayarlarını otomatik uygular.

## Tam SVG Örneği

```xml
<svg viewBox="0 0 300 100">
  <!-- 3 motor durum göstergesi -->
  <g transform="translate(30,50)">
    <circle id="motor1_led" r="15" fill="#ccc"/>
    <text y="35" text-anchor="middle" font-size="11">Motor 1</text>
  </g>
  <g transform="translate(150,50)">
    <circle id="motor2_led" r="15" fill="#ccc"/>
    <text y="35" text-anchor="middle" font-size="11">Motor 2</text>
  </g>
  <g transform="translate(270,50)">
    <circle id="motor3_led" r="15" fill="#ccc"/>
    <text y="35" text-anchor="middle" font-size="11">Motor 3</text>
  </g>
</svg>
```

Her `motor*_led` için Color element:
- Expression Type: Switch
- Expression: `Motor1_Status` (Tag)
- Switch: `true → #00cc00 | false → #ff0000`
