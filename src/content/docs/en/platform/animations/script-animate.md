---
title: "Script & Animate"
description: "Özel JavaScript, CSS animasyon, yetki kontrolü, 3D, QR kod"
sidebar:
  order: 20
---

## Script

**Script**, SVG öğesiyle ilişkilendirilmiş özel JavaScript kodu çalıştırır. Her animation döngüsünde (duration) periyodik olarak çalışır. Diğer animation tipleriyle yapılamayan karmaşık mantık için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Script |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Expression |

### Expression Örnekleri

**Birden fazla DOM öğesini güncelleme:**
```javascript
var power = ins.getVariableValue("ActivePower_kW").value;
var voltage = ins.getVariableValue("Voltage_V").value;
var current = voltage > 0 ? (power * 1000 / voltage) : 0;

// Hesaplanan akımı göster
var el = document.getElementById("calc_current");
el.textContent = current.toFixed(1) + " A";

// Rengini ayarla
el.setAttribute("fill", current > 50 ? "#ff0000" : "#00cc00");
```

**Dinamik SVG oluşturma:**
```javascript
var values = ins.getVariableValues(["Temp1", "Temp2", "Temp3", "Temp4"]);
var container = document.getElementById("temp_bars");
container.innerHTML = "";

var names = ["Temp1", "Temp2", "Temp3", "Temp4"];
for (var i = 0; i < names.length; i++) {
    var val = values[names[i]].value;
    var height = val * 2; // 0-100 → 0-200px
    var color = val > 70 ? "#ff0000" : val > 50 ? "#ff8800" : "#00cc00";

    var rect = document.createElementNS("http://www.w3.org/2000/svg", "rect");
    rect.setAttribute("x", i * 50 + 10);
    rect.setAttribute("y", 200 - height);
    rect.setAttribute("width", 35);
    rect.setAttribute("height", height);
    rect.setAttribute("fill", color);
    container.appendChild(rect);
}
```

:::tip
Basit binding'ler için Get, Color, Bar gibi hazır tipleri tercih edin. Script her döngüde çalışır ve DOM manipülasyonu performansı etkiler.
:::

### FormScript

**FormScript**, form tabanlı script bileşenidir. Kullanıcıdan birden fazla girdi alarak toplu işlem yapar.

| Kullanım | Örnek |
|----------|-------|
| Tarif değiştirme | Birden fazla setpoint'i tek formda güncelle |
| Rapor parametreleri | Tarih aralığı + değişken seçimi ile rapor oluştur |
| Toplu parametre ayarı | PID parametrelerini (P, I, D) tek formda düzenle |

---

## Animate (CSS/SVG Animasyon)

**Animate**, koşula göre CSS animation veya SVG SMIL animasyonunu başlatır/durdurur. Dönme, titreşim, pulse, yanıp sönme gibi görsel efektler için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Animate |
| **Uygun SVG Öğeleri** | `<g>`, `<path>`, `<circle>` |
| **Expression Type** | Tag (Boolean), Expression |

### SVG + CSS Örneği — Dönen Fan

```xml
<style>
  @keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }
  .spinning { animation: spin 1s linear infinite; transform-origin: center; }
</style>

<g id="fan_blades" transform="translate(100,100)">
  <line x1="-30" y1="0" x2="30" y2="0" stroke="#333" stroke-width="4"/>
  <line x1="0" y1="-30" x2="0" y2="30" stroke="#333" stroke-width="4"/>
  <line x1="-21" y1="-21" x2="21" y2="21" stroke="#333" stroke-width="4"/>
  <line x1="21" y1="-21" x2="-21" y2="21" stroke="#333" stroke-width="4"/>
</g>
```

Expression: `Fan_Running` (Tag, Boolean)
- `true` → `fan_blades`'e `.spinning` class eklenir → fan döner
- `false` → class kaldırılır → fan durur

---

## Access (Yetki Kontrolü)

**Access**, kullanıcı yetkisine göre SVG öğesini gösterir veya gizler. Güvenlik gerektiren kontrol butonlarını yalnızca yetkili kullanıcılara göstermek için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Access |
| **Uygun SVG Öğeleri** | `<g>` (kontrol butonlarını grupla) |
| **Expression** | Gerekli yetki adı |

### Örnek

```xml
<!-- Yalnızca SET_VARIABLE_VALUE yetkisi olanlar görebilir -->
<g id="control_buttons">
  <rect id="btn_start" ... />
  <rect id="btn_stop" ... />
</g>
```

- `control_buttons` → **Access**, Expression: `SET_VARIABLE_VALUE`
- Yetkisi olmayan kullanıcılar bu grubu hiç göremez

---

## Three (3D Görselleştirme)

**Three**, Three.js tabanlı 3D model gömme sağlar.

| Alan | Değer |
|------|-------|
| **Type** | Three |
| **Uygun SVG Öğeleri** | `<rect>` (foreignObject) |

Kullanım: 3D tesis modeli, ekipman görselleştirme, sanal tur.

---

## QRCodeGeneration (QR Kod Oluşturma)

**QRCodeGeneration**, değişken değerinden veya metinden QR kod oluşturur.

| Alan | Değer |
|------|-------|
| **Type** | QRCodeGeneration |
| **Expression Type** | Expression, Text |

```javascript
// Cihaz bilgilerini QR koda çevir
var sn = ins.getVariableValue("Serial_Number").value;
return "https://inscada.com/device/" + sn;
```

Kullanım: Cihaz kimlik kodu, bakım formu URL'si, envanter etiketi.

## QRCodeScan (QR Kod Okuma)

**QRCodeScan**, kamera ile QR kod tarayıcı bileşeni açar. Taranan veriyi değişkene veya script'e aktarır.

Kullanım: Mobil cihazdan cihaz tanıma, bakım kaydı girişi, saha personeli kimlik doğrulama.
