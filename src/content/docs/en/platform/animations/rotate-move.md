---
title: "Rotate & Move"
description: "Döndürme ve kaydırma animasyonları"
sidebar:
  order: 13
---

## Rotate (Döndürme)

**Rotate**, bir SVG öğesini değere göre döndürür. Gösterge ibresi, vana pozisyonu, rüzgar yönü, kompas gibi dairesel gösterimlerde kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Rotate |
| **Uygun SVG Öğeleri** | `<g>`, `<path>`, `<line>`, `<rect>` |
| **Expression Type** | Tag, Expression |

### Yapılandırma (Props)

| Özellik | Açıklama |
|---------|----------|
| **cx** | Döndürme merkezi X koordinatı |
| **cy** | Döndürme merkezi Y koordinatı |
| **minAngle** | Minimum açı (derece) |
| **maxAngle** | Maksimum açı (derece) |
| **minValue** | Minimum değer |
| **maxValue** | Maksimum değer |

### SVG Hazırlığı — Analog Gösterge

```xml
<svg viewBox="0 0 200 200">
  <!-- Kadran arka planı -->
  <circle cx="100" cy="100" r="90" fill="none" stroke="#ddd" stroke-width="4"/>

  <!-- Kadran çizgileri -->
  <line x1="100" y1="15" x2="100" y2="25" stroke="#333" stroke-width="2"/>

  <!-- İbre -->
  <g id="gauge_needle">
    <line x1="100" y1="100" x2="100" y2="20"
          stroke="#e74c3c" stroke-width="3" stroke-linecap="round"/>
    <circle cx="100" cy="100" r="5" fill="#e74c3c"/>
  </g>

  <!-- Değer metni -->
  <text id="gauge_value" x="100" y="160"
        text-anchor="middle" font-size="18" fill="#333">--</text>
</svg>
```

### Yapılandırma

- `gauge_needle` → **Rotate**
  - Props: cx=100, cy=100, minAngle=-135, maxAngle=135, minValue=0, maxValue=100
  - Expression Type: Tag → `Temperature_C`
- `gauge_value` → **Get**
  - Expression: `ins.getVariableValue("Temperature_C").value.toFixed(1) + " °C"`

Sonuç: Sıcaklık 0°C'de ibre sol alt (-135°), 100°C'de sağ alt (135°).

### Vana Pozisyonu Örneği

```javascript
// 0 = kapalı (0°), 100 = açık (90°)
var pos = ins.getVariableValue("Valve_Position").value;
return pos * 0.9; // 0-90 derece
```

---

## Move (Kaydırma)

**Move**, bir SVG öğesini X ve/veya Y ekseninde değere göre kaydırır. Seviye göstergesi, konveyör pozisyonu, asansör gibi lineer hareket animasyonlarında kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Move |
| **Uygun SVG Öğeleri** | `<g>`, `<rect>`, `<circle>`, `<image>` |
| **Expression Type** | Tag, Expression |

### Yapılandırma (Props)

| Özellik | Açıklama |
|---------|----------|
| **axis** | Hareket ekseni: `x` veya `y` |
| **minPos** | Minimum pozisyon (piksel) |
| **maxPos** | Maksimum pozisyon (piksel) |
| **minValue** | Minimum değer |
| **maxValue** | Maksimum değer |

### Asansör Örneği

```xml
<svg viewBox="0 0 100 400">
  <!-- Kuyu -->
  <rect x="20" y="10" width="60" height="380" fill="none" stroke="#999"/>
  <!-- Kabin -->
  <g id="elevator_cabin">
    <rect x="25" y="0" width="50" height="40" fill="#3498db" rx="3"/>
  </g>
</svg>
```

- `elevator_cabin` → **Move**
  - Props: axis=y, minPos=10, maxPos=350, minValue=0, maxValue=10
  - Expression Type: Tag → `Floor_Position`
  - Kat 0'da kabin en altta, kat 10'da en üstte

### Seviye Göstergesi Örneği

```javascript
// Tank seviyesi 0-100% → Y pozisyon 200-0 (ters yön)
var level = ins.getVariableValue("Tank_Level").value;
return 200 - (level * 2);
```
