---
title: "Veri Gösterimi"
description: "Get, Color, Bar, Opacity, Visibility, Rotate, Move, Scale, Blink, Pipe, Tooltip, Image, AlarmIndication"
sidebar:
  order: 1
---

Bu sayfada değişken değerlerini ekranda görselleştirmek için kullanılan animation element tipleri anlatılmaktadır.

## Get (Değer Gösterme)

SVG öğesinin metin içeriğini değişken değeri ile günceller. En sık kullanılan animation tipidir.

| Alan | Değer |
|------|-------|
| **Type** | Get |
| **DOM ID** | SVG text öğesinin id'si |
| **Expression Type** | Tag (değişken adı) veya Expression (formül) |

```xml
<text id="power_display" font-size="24">--</text>
```

Expression örnekleri:
- **Tag:** `ActivePower_kW` → doğrudan değişken değerini yazar
- **Expression:** `value.toFixed(1) + " kW"` → formatlı gösterim

## Color (Renk Değiştirme)

SVG öğesinin `fill` veya `stroke` rengini değere göre değiştirir.

| Alan | Değer |
|------|-------|
| **Type** | Color |
| **DOM ID** | SVG rect, circle, path vb. |
| **Expression Type** | Expression veya Switch |

```xml
<circle id="status_led" r="10" fill="#cccccc"/>
```

**Switch expression ile:**
```
true → #00cc00
false → #ff0000
```

**Expression ile:**
```javascript
if (value > 80) return '#ff0000';
else if (value > 60) return '#ff8800';
else return '#00cc00';
```

## Bar (Çubuk Grafik)

Bir dikdörtgenin yüksekliğini veya genişliğini değere orantılı olarak ayarlar. Tank seviyesi, ilerleme çubuğu gibi gösterimlerde kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Bar |
| **DOM ID** | SVG rect |
| **Props** | Yön, min/max değerler |

```xml
<rect id="level_bar" x="10" y="100" width="30" height="0" fill="#3498db"/>
```

Değer 0-100 arasında → rect yüksekliği 0-200px arasında ölçeklenir.

## Opacity (Saydamlık)

Öğenin saydamlığını değere göre ayarlar (0.0 = tamamen saydam, 1.0 = tamamen opak).

| Alan | Değer |
|------|-------|
| **Type** | Opacity |
| **DOM ID** | Herhangi bir SVG öğesi |

Kullanım: Haberleşme durumuna göre cihaz sembolünü soluklaştırma.

## Visibility (Göster/Gizle)

Boolean değere göre SVG öğesini gösterir veya gizler (`display: none`).

| Alan | Değer |
|------|-------|
| **Type** | Visibility |
| **DOM ID** | SVG group veya öğe |
| **Expression Type** | Tag (Boolean değişken) veya Expression |

```xml
<g id="alarm_icon" display="none">
  <polygon points="..." fill="red"/>
</g>
```

`GridStatus = true` → görünür, `false` → gizli

## Rotate (Döndürme)

SVG öğesini değere göre döndürür. Gösterge ibresi, vana pozisyonu, rüzgar yönü gibi gösterimlerde kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Rotate |
| **DOM ID** | SVG group |
| **Props** | Döndürme merkezi (cx, cy), min/max açı |

```xml
<g id="needle" transform="rotate(0, 60, 60)">
  <line x1="60" y1="60" x2="60" y2="15" stroke="red" stroke-width="2"/>
</g>
```

Değer 0-100 → açı 0°-270° arasında ölçeklenir.

## Move (Kaydırma)

SVG öğesini X veya Y ekseninde değere göre kaydırır. Seviye göstergesi, konveyör pozisyonu gibi animasyonlarda kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Move |
| **DOM ID** | SVG group |
| **Props** | Yön (X/Y), mesafe aralığı |

## Scale (Ölçekleme)

SVG öğesini değere orantılı olarak büyütür/küçültür.

| Alan | Değer |
|------|-------|
| **Type** | Scale |
| **DOM ID** | SVG group |

## Blink (Yanıp Sönme)

Koşul sağlandığında SVG öğesini yanıp söndürür. Alarm durumu, dikkat gerektiren göstergeler için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Blink |
| **DOM ID** | Herhangi bir SVG öğesi |
| **Expression Type** | Tag veya Expression (Boolean sonuç) |

```xml
<circle id="alarm_blink" r="12" fill="red"/>
```

`value = true` → yanıp sönme başlar, `false` → durur.

## Pipe (Akış Animasyonu)

Boru hatları veya kablolarda akış yönünü gösteren animasyon. Çizgi üzerinde hareket eden tire deseni oluşturur.

| Alan | Değer |
|------|-------|
| **Type** | Pipe |
| **DOM ID** | SVG path veya line |
| **Expression Type** | Tag (Boolean — akış var/yok) |

`value = true` → animasyonlu akış, `false` → durağan çizgi.

## Tooltip (Bilgi Balonu)

SVG öğesi üzerine gelindiğinde bilgi balonu gösterir. Detay bilgi, ek parametreler için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Tooltip |
| **DOM ID** | Herhangi bir SVG öğesi |
| **Expression** | Tooltip içeriği (HTML destekler) |

## Image (Resim Değiştirme)

Değere göre SVG image öğesinin `href` kaynağını değiştirir. Durum bazlı farklı ikonlar göstermek için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Image |
| **DOM ID** | SVG image |

**Switch expression ile:**
```
0 → /images/motor-off.png
1 → /images/motor-on.png
2 → /images/motor-fault.png
```

## AlarmIndication (Alarm Göstergesi)

Bir alarm grubunun veya alarmın durumunu renk kodlarıyla gösterir. Alarm grubu renk ayarlarını (OnNoAck, OnAck, OffNoAck, OffAck) kullanır.

| Alan | Değer |
|------|-------|
| **Type** | AlarmIndication |
| **DOM ID** | SVG rect, circle vb. |
| **Expression Type** | Alarm |

Otomatik olarak alarm grubunun tanımlı renklerini uygular — yanıp sönme, sabit renk vb.
