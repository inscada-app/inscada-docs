---
title: "Bar & Scale"
description: "Çubuk gösterge ve ölçekleme animasyonları"
sidebar:
  order: 12
---

## Bar (Çubuk Gösterge)

**Bar**, bir SVG dikdörtgeninin yüksekliğini veya genişliğini değere orantılı olarak değiştirir. Tank seviyesi, ilerleme çubuğu, yük göstergesi gibi gösterimlerde kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Bar |
| **Uygun SVG Öğeleri** | `<rect>` |
| **Expression Type** | Tag, Expression |

### Yapılandırma (Props)

| Özellik | Açıklama |
|---------|----------|
| **direction** | Yön: `vertical` (aşağıdan yukarı) veya `horizontal` (soldan sağa) |
| **min** | Minimum değer (bu değerde bar boş) |
| **max** | Maksimum değer (bu değerde bar dolu) |
| **fillColor** | Dolgu rengi |

### SVG Hazırlığı — Dikey Bar

```xml
<!-- Tank çerçevesi -->
<rect x="40" y="20" width="40" height="200" fill="none" stroke="#666" stroke-width="2"/>
<!-- Seviye bar'ı (başlangıçta boş) -->
<rect id="tank_level" x="42" y="220" width="36" height="0" fill="#3498db"/>
```

### Expression Örnekleri

**Tag:**
```
Temperature_C
```
Props: `min=0, max=100, direction=vertical`
→ 50°C'de bar %50 dolu

**Expression ile renk değişimi:**
```javascript
var val = ins.getVariableValue("Temperature_C").value;
// Değeri döndür, renk ayrıca Color element ile ayarlanabilir
return val;
```

### Tam Örnek — Yatay İlerleme Çubuğu

```xml
<svg viewBox="0 0 400 60">
  <!-- Arka plan -->
  <rect x="10" y="20" width="300" height="20" fill="#e0e0e0" rx="3"/>
  <!-- İlerleme -->
  <rect id="progress_bar" x="10" y="20" width="0" height="20" fill="#2196F3" rx="3"/>
  <!-- Yüzde metni -->
  <text id="progress_text" x="320" y="35" font-size="14">0%</text>
</svg>
```

- `progress_bar` → Bar, Tag: `PowerFactor`, Props: min=0, max=1, horizontal
- `progress_text` → Get, Expression: `(ins.getVariableValue("PowerFactor").value * 100).toFixed(0) + "%"`

---

## Scale (Ölçekleme)

**Scale**, bir SVG öğesini veya grubunu değere göre büyütür veya küçültür. `transform: scale()` CSS özelliğini kullanır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Scale |
| **Uygun SVG Öğeleri** | `<g>`, `<rect>`, `<circle>`, `<path>` |
| **Expression Type** | Tag, Expression |

### Expression Örnekleri

```javascript
// 0-1000 kW aralığını 0.5-2.0 scale'e dönüştür
var power = ins.getVariableValue("ActivePower_kW").value;
return 0.5 + (power / 1000) * 1.5;
```

### Kullanım Senaryoları

- Üretim miktarına göre büyüyen/küçülen sembol
- Dinamik boyutlu gauge ibresi
- Alarm durumunda büyüyen uyarı ikonu
