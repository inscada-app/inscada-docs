---
title: "Set, Button & Click"
description: "Değer yazma, buton ve tıklama/fare olay kontrol animasyonları"
sidebar:
  order: 17
---

Bu sayfadaki animation tipleri, operatörün SVG ekran üzerinden değişkenlere değer yazmasını ve etkileşimde bulunmasını sağlar.

:::caution
Kontrol tipleri saha cihazlarına komut gönderir. Kullanıcının `SET_VARIABLE_VALUE` yetkisi gereklidir.
:::

## Set (Değer Yazma)

**Set**, SVG öğesine tıklandığında hedef değişkene değer yazar. 6 farklı etkileşim modu sunar — basit otomatik yazmadan kaydırıcılı ayarlamaya kadar.

| Alan | Değer |
|------|-------|
| **Type** | Set (Click sekmesi altında) |
| **Uygun SVG Öğeleri** | Tümü (tıklanabilir herhangi bir öğe) |

### Yapılandırma Alanları

| Alan | Açıklama |
|------|----------|
| **Variable** | Hedef değişken seçimi |
| **Default** | Yazılacak varsayılan değer |
| **Description** | Onay diyaloğunda gösterilecek açıklama metni |
| **Set Type** | Etkileşim modu (aşağıdaki tabloya bakın) |
| **Notify** | Değer yazıldığında bildirim göster |
| **Log** | Değer yazma işlemini logla |
| **Hide Info Popup** | Bilgi popup'ını gizle |

### Set Type (Etkileşim Modları)

| Mod | Açıklama |
|-----|----------|
| **AUTO** | Tıklandığında değeri **anında yazar** — diyalog göstermez |
| **YES/NO** | Onay diyaloğu gösterir: "Emin misiniz?" → Evet / Hayır |
| **MANUAL** | Protokole özel kontrol arayüzü açar (MODBUS register yazma, DNP3 komut vb.) |
| **SLIDER** | Kaydırıcılı değer ayarlama diyaloğu açar |
| **TOGGLE** | Toggle (aç/kapa) switch diyaloğu gösterir |
| **AUTO TOGGLE** | Mevcut değeri **anında tersine çevirir** — diyalog göstermez (true↔false) |

#### Slider Modu Ek Alanları

SLIDER seçildiğinde ek alanlar görünür:

| Alan | Açıklama |
|------|----------|
| **Min Value** | Kaydırıcı minimum değeri |
| **Max Value** | Kaydırıcı maksimum değeri |
| **Step** | Adım büyüklüğü |
| **Horizontal** | Yatay kaydırıcı |
| **On Drop** | Bırakıldığında yaz (sürekli yerine) |

### Şifre Koruması

Set element'ine şifre koruması eklenebilir — operatör değer yazmadan önce şifre girmelidir:

| Alan | Açıklama |
|------|----------|
| **Password** | Şifre koruması etkinleştir |
| **Password Value** | Sabit şifre değeri |
| **Password Variable** | Şifreyi bir değişkenden oku |
| **Numpad** | Şifre girişinde sayısal tuş takımı göster |

### SVG Örneği

```xml
<g cursor="pointer">
  <rect id="btn_start" x="10" y="10" width="100" height="40" rx="5" fill="#4CAF50"/>
  <text x="60" y="35" text-anchor="middle" fill="white" font-size="14"
        pointer-events="none">START</text>
</g>
```

- Set Type: **AUTO** → Tıklandığında anında `Motor_Run = true`
- Set Type: **YES/NO** → "Motor başlatılsın mı?" onay diyaloğu

---

## Button (Buton Bileşeni)

**Button**, tamamen özelleştirilebilir HTML buton bileşeni oluşturur. Set'ten farkı: görsel stil sistemi ve tıklamada JavaScript kodu çalıştırma desteği.

| Alan | Değer |
|------|-------|
| **Type** | Button |
| **Uygun SVG Öğeleri** | `<rect>` (foreignObject + HTML button olarak render edilir) |

### Yapılandırma Alanları

#### Stil Ayarları

| Alan | Açıklama |
|------|----------|
| **Group** | Buton stil grubu (Alpha, Beta, Gamma, Delta, ... , Stock) |
| **Type** | Buton tipi (Primary, Secondary, Tertiary, Black, Dark, Gray, Light) |
| **Size** | Buton boyutu (Small, Regular, Medium, Large) |
| **Button Name** | Buton üzerindeki metin |

#### Özel Stil

| Alan | Açıklama |
|------|----------|
| **Width** | Özel genişlik (px) |
| **Height** | Özel yükseklik (px) |
| **Font Color** | Metin rengi |
| **Font Size** | Yazı boyutu (px) |
| **Font Weight** | Yazı kalınlığı |
| **Background** | Arka plan rengi |

### onClick Expression

Butona tıklandığında çalışacak JavaScript kodu:

```javascript
// Değer yazma
ins.setVariableValue("Motor_Run", {value: true});
ins.notify("success", "Kontrol", "Motor başlatıldı");
```

```javascript
// PDF dışa aktarma
return { type: 'exportPdf' };
```

```javascript
// Başka animation'a geç
return { type: 'animation', clickUrl: 'Motor_Detail' };
```

### Return Değerleri

Button expression'dan döndürülen özel komutlar:

| Return | Açıklama |
|--------|----------|
| `{ type: 'animation', clickUrl: 'name' }` | Başka animation'a geç |
| `{ type: 'popupAnimation', clickUrl: 'name' }` | Popup animation aç |
| `{ type: 'parentAnimation' }` | Üst animation'a dön |
| `{ type: 'popupClose' }` | Popup'ı kapat |
| `{ type: 'exportPdf' }` | Mevcut ekranı PDF olarak dışa aktar |
| `{ type: 'printCurrentSvg' }` | Mevcut SVG'yi yazdır |
| `{ type: 'changeLang', clickUrl: 'tr' }` | Dil değiştir |
| `{ type: 'map' }` | Harita görünümüne geç |
| `{ type: 'systemPage', clickUrl: 'page' }` | Sistem sayfasına git |

---

## Click (Tıklama Olayı)

**Click**, SVG öğesine tıklandığında JavaScript kodu çalıştırır. Set'ten farkı: sabit değer yazmak yerine tamamen programatik işlem yapabilir.

| Alan | Değer |
|------|-------|
| **Type** | Click |
| **Uygun SVG Öğeleri** | Tümü |

### Expression Tipleri

Click, birden fazla expression tipi destekler:

| Tip | Açıklama |
|-----|----------|
| **EXPRESSION** | JavaScript kodu çalıştır |
| **SET** | Değişkene değer yaz (Set ile aynı) |
| **ANIMATION** | Başka animation'a geç |
| **ANIMATION POPUP** | Popup animation aç |
| **URL** | URL aç (yeni pencere) |
| **ALARM** | Alarm Monitor veya Alarm History göster |
| **SYSTEM PAGE** | Sistem sayfasına git |
| **COLLECTION** | Birden fazla işlem |

### Yapılandırma

| Alan | Açıklama |
|------|----------|
| **Outline Thickness** | Hover'da gösterilen kırmızı çerçeve kalınlığı |
| **Parent Dom** | Üst DOM öğesini kullan |

### Expression Örnekleri

```javascript
// Onaylı değer yazma
if (confirm('Motor başlatılsın mı?')) {
    ins.setVariableValue('Motor_Run', {value: true});
    ins.writeLog("INFO", "Control", "Motor started by operator");
}
```

```javascript
// Toggle
var current = ins.getVariableValue("Pump_Running").value;
ins.setVariableValue("Pump_Running", {value: !current});
```

```javascript
// Toplu komut
if (confirm('Tüm pompalar durdurulsun mu?')) {
    ins.setVariableValues({
        "Pump1_Run": {value: false},
        "Pump2_Run": {value: false},
        "Pump3_Run": {value: false}
    });
    ins.notify("warning", "Kontrol", "Tüm pompalar durduruldu");
}
```

```javascript
// Başka animation'a geçiş
return { type: 'animation', clickUrl: 'Motor_Detail' };
```

---

## MouseDown / MouseUp (Basılı Tutma)

**MouseDown** ve **MouseUp**, fare butonuna basıldığında ve bırakıldığında ayrı eylemler tetikler. **Jog** (anlık hareket) kontrolleri için idealdir.

| Tip | Tetikleme |
|-----|-----------|
| **MouseDown** | Fare butonu basıldığında |
| **MouseUp** | Fare butonu bırakıldığında |

Click ile aynı expression tiplerini ve yapılandırma alanlarını destekler.

### Jog Kontrolü Örneği

```xml
<rect id="btn_jog_forward" width="80" height="40" fill="#2196F3" cursor="pointer"/>
```

- `btn_jog_forward` → **MouseDown**: `ins.setVariableValue("Jog_Forward", {value: true})`
- `btn_jog_forward` → **MouseUp**: `ins.setVariableValue("Jog_Forward", {value: false})`

Basılı tuttuğunuz sürece motor ileri hareket eder, bırakınca durur.

---

## MouseOver (Fare Üzerine Gelme)

**MouseOver**, fare öğe üzerine geldiğinde tetiklenir. Click ile aynı expression tiplerini destekler.

| Alan | Değer |
|------|-------|
| **Type** | MouseOver |
| **Uygun SVG Öğeleri** | Tümü |

Tooltip'ten farkı: JavaScript kodu çalıştırabilir, değişken okuyabilir/yazabilir, başka öğeleri güncelleyebilir, animation açabilir.

---

## Hover Geri Bildirimi

Tüm tıklanabilir element'ler (Set, Click, MouseDown, MouseUp, MouseOver) hover'da kırmızı çerçeve ile görsel geri bildirim sağlar. Çerçeve kalınlığı **Outline Thickness** alanından ayarlanır. Bu sayede operatör hangi objelerin tıklanabilir olduğunu anlayabilir.
