---
title: "Kontrol & Etkileşim"
description: "Set, Slider, Input, Button, Click ve fare olay animation tipleri"
sidebar:
  order: 4
---

Bu sayfadaki animation tipleri, operatörün ekran üzerinden değişkenlere değer yazmasını ve etkileşimde bulunmasını sağlar.

:::caution
Kontrol tipleri değişkenlere değer yazar. Kullanıcının `SET_VARIABLE_VALUE` yetkisine sahip olması gerekir.
:::

## Set (Değer Yazma)

SVG öğesine tıklandığında belirtilen değişkene sabit bir değer yazar. Motor start/stop, vana aç/kapa gibi temel kontroller için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Set |
| **DOM ID** | Tıklanabilir SVG öğesi |
| **Expression Type** | Set |
| **Expression** | Yazılacak değer ve hedef değişken |

```xml
<!-- Start butonu -->
<rect id="btn_start" x="10" y="10" width="80" height="30"
      rx="5" fill="#4CAF50" cursor="pointer"/>
<text x="50" y="30" text-anchor="middle" fill="white">START</text>
```

Tıklandığında: `Motor_Run → {value: true}`

## Slider (Kaydırıcı)

Sürüklenebilir kaydırıcı ile analog değer ayarlamak için kullanılır. Setpoint, hız ayarı, sıcaklık hedefi gibi kontroller.

| Alan | Değer |
|------|-------|
| **Type** | Slider |
| **DOM ID** | SVG group |
| **Props** | Min, max, step, yön (horizontal/vertical) |

Operatör kaydırıcıyı sürükleyerek değeri değiştirir → değişkene otomatik yazılır.

## Input (Giriş Alanı)

Metin veya sayı giriş alanı. Operatör klavyeden değer girer.

| Alan | Değer |
|------|-------|
| **Type** | Input |
| **DOM ID** | SVG foreignObject |
| **Props** | Tip (text/number), min, max, placeholder |

```xml
<foreignObject id="setpoint_input" x="10" y="50" width="120" height="30"/>
```

Kullanım: Setpoint değeri girme, tarif adı yazma, parametre düzenleme.

## Button (Buton)

Webix buton bileşeni olarak çalışır. Tıklandığında değer yazma veya script çalıştırma yapabilir.

| Alan | Değer |
|------|-------|
| **Type** | Button |
| **DOM ID** | SVG foreignObject |
| **Expression Type** | Button |
| **Props** | Buton metni, stil, eylem |

## Click (Tıklama Olayı)

SVG öğesine tıklandığında özel bir JavaScript fonksiyonu çalıştırır. Set'ten farkı: sabit değer yazmak yerine programatik işlem yapabilir.

| Alan | Değer |
|------|-------|
| **Type** | Click |
| **DOM ID** | Herhangi bir SVG öğesi |
| **Expression Type** | Expression |

```javascript
// Tıklanınca onay iste ve değer yaz
if (confirm('Motor başlatılsın mı?')) {
    ins.setVariableValue('Motor_Run', {value: true});
}
```

## MouseDown / MouseUp (Basılı Tutma)

Mouse butonuna basıldığında ve bırakıldığında ayrı eylemler tetikler. Jog (anlık hareket) kontrolleri için idealdir.

| Tip | Tetikleme |
|-----|-----------|
| **MouseDown** | Mouse basıldığında |
| **MouseUp** | Mouse bırakıldığında |

Kullanım: Jog ileri/geri, basılı tutunca ON — bırakınca OFF.

## MouseOver (Fare Üzerine Gelme)

Fare öğe üzerine geldiğinde eylem tetikler. Tooltip gösterme, detay bilgi çekme gibi işlemler.

| Alan | Değer |
|------|-------|
| **Type** | MouseOver |
| **DOM ID** | Herhangi bir SVG öğesi |
| **Expression Type** | Expression |
