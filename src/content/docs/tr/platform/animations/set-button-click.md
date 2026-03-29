---
title: "Set, Button & Click"
description: "Değer yazma, buton ve tıklama kontrol animasyonları"
sidebar:
  order: 17
---

Bu sayfadaki animation tipleri, operatörün SVG ekran üzerinden değişkenlere değer yazmasını sağlar.

:::caution
Kontrol tipleri saha cihazlarına komut gönderir. Kullanıcının `SET_VARIABLE_VALUE` yetkisi gereklidir.
:::

## Set (Değer Yazma)

**Set**, SVG öğesine tıklandığında hedef değişkene önceden tanımlı bir değer yazar. Motor start/stop, vana aç/kapa gibi basit kontroller için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Set |
| **Uygun SVG Öğeleri** | Tümü (tıklanabilir herhangi bir öğe) |
| **Expression Type** | Set |

### SVG Hazırlığı

```xml
<g cursor="pointer">
  <!-- START butonu -->
  <rect id="btn_start" x="10" y="10" width="100" height="40"
        rx="5" fill="#4CAF50"/>
  <text x="60" y="35" text-anchor="middle" fill="white"
        font-size="14" pointer-events="none">START</text>
</g>

<g cursor="pointer">
  <!-- STOP butonu -->
  <rect id="btn_stop" x="130" y="10" width="100" height="40"
        rx="5" fill="#f44336"/>
  <text x="180" y="35" text-anchor="middle" fill="white"
        font-size="14" pointer-events="none">STOP</text>
</g>
```

### Yapılandırma

- `btn_start` → **Set**, hedef: `Motor_Run`, değer: `{value: true}`
- `btn_stop` → **Set**, hedef: `Motor_Run`, değer: `{value: false}`

Tıklandığında doğrudan `ins.setVariableValue()` çağrılır.

### Onay Diyaloğu

Props'ta `confirm: true` ayarı ile tıklama öncesi onay penceresi gösterilebilir:

```
"Motor başlatılsın mı?" → Evet / Hayır
```

---

## Button (Buton)

**Button**, Webix buton bileşeni olarak çalışır. Set'ten farkı: görsel olarak Webix buton stiline sahiptir ve daha karmaşık eylemler tetikleyebilir.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Button |
| **Uygun SVG Öğeleri** | `<rect>` (foreignObject) |
| **Expression Type** | Button |

### Yapılandırma (Props)

| Özellik | Açıklama |
|---------|----------|
| **label** | Buton metni |
| **css** | CSS sınıfı (renk/stil) |
| **action** | Tıklama eylemi |

---

## Click (Tıklama Olayı)

**Click**, SVG öğesine tıklandığında özel JavaScript kodu çalıştırır. Set'ten farkı: sabit değer yazmak yerine programatik işlem yapabilir — onay diyaloğu, hesaplama, birden fazla değişkene yazma gibi.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Click |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Expression |

### Expression Örnekleri

**Onaylı değer yazma:**
```javascript
if (confirm('Motor başlatılsın mı?')) {
    ins.setVariableValue('Motor_Run', {value: true});
    ins.writeLog("INFO", "Control", "Motor started by operator");
}
```

**Toggle (aç/kapa):**
```javascript
var current = ins.getVariableValue("Pump_Running").value;
ins.setVariableValue("Pump_Running", {value: !current});
```

**Toplu komut:**
```javascript
if (confirm('Tüm pompalar durdurulsun mu?')) {
    ins.setVariableValues({
        "Pump1_Run": {value: false},
        "Pump2_Run": {value: false},
        "Pump3_Run": {value: false}
    });
    ins.notify("warning", "Kontrol", "Tüm pompalar durduruldu");
}
```

**Başka animation'a geçiş:**
```javascript
// Tıklanan motora göre detay ekranı aç
var motorId = 3;
// Open animation tipi de kullanılabilir ama Click ile
// parametre geçişi daha esnek
```

---

## MouseDown / MouseUp (Basılı Tutma)

**MouseDown** ve **MouseUp**, fare butonuna basıldığında ve bırakıldığında ayrı eylemler tetikler. **Jog** (anlık hareket) kontrolleri için idealdir.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | MouseDown / MouseUp |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Expression |

### Jog Kontrolü Örneği

```xml
<rect id="btn_jog_forward" width="80" height="40" fill="#2196F3" cursor="pointer"/>
<text x="40" y="25" text-anchor="middle" fill="white">JOG →</text>
```

- `btn_jog_forward` → **MouseDown**: `ins.setVariableValue("Jog_Forward", {value: true})`
- `btn_jog_forward` → **MouseUp**: `ins.setVariableValue("Jog_Forward", {value: false})`

Basılı tuttuğunuz sürece motor ileri hareket eder, bırakınca durur.

---

## MouseOver (Fare Üzerine Gelme)

**MouseOver**, fare öğe üzerine geldiğinde eylem tetikler.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | MouseOver |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Expression |

Tooltip'ten farkı: JavaScript kodu çalıştırabilir, değişken okuyabilir, başka öğeleri güncelleyebilir.
