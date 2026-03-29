---
title: "Opacity, Visibility & Blink"
description: "Saydamlık, göster/gizle ve yanıp sönme animasyonları"
sidebar:
  order: 14
---

## Opacity (Saydamlık)

**Opacity**, bir SVG öğesinin saydamlığını değere göre ayarlar. Haberleşme durumuna göre cihaz sembolünü soluklaştırma, güç durumuna göre bölge parlaklığı gibi efektler için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Opacity |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Tag, Expression |

### Expression Örnekleri

```javascript
// Bağlantı durumuna göre: bağlı=opak, bağlı değil=soluk
var status = ins.getConnectionStatus("MODBUS-PLC");
return status === "Connected" ? 1.0 : 0.3;
```

```javascript
// Değere orantılı saydamlık (0-100 → 0.2-1.0)
var val = ins.getVariableValue("Signal_Strength").value;
return 0.2 + (val / 100) * 0.8;
```

---

## Visibility (Göster/Gizle)

**Visibility**, bir SVG öğesini koşula göre gösterir veya tamamen gizler (`display: none`). Opacity'den farkı: kademeli değil, ya tamamen görünür ya tamamen gizli.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Visibility |
| **Uygun SVG Öğeleri** | Tümü (özellikle `<g>` grupları) |
| **Expression Type** | Tag (Boolean), Expression |

### SVG Hazırlığı

```xml
<!-- Alarm ikonu — normalde gizli -->
<g id="alarm_warning" display="none">
  <polygon points="100,10 120,50 80,50" fill="#ff0000"/>
  <text x="100" y="43" text-anchor="middle" fill="white" font-size="20">!</text>
</g>
```

### Expression Örnekleri

**Boolean değişken — Tag:**
```
GridStatus
```
`true` → görünür, `false` → gizli

**Eşik koşulu — Expression:**
```javascript
var temp = ins.getVariableValue("Temperature_C").value;
return temp > 70; // 70°C üzerinde uyarı ikonu görünsün
```

**Birden fazla koşul:**
```javascript
var power = ins.getVariableValue("ActivePower_kW").value;
var status = ins.getVariableValue("GridStatus").value;
return power > 500 && status; // hem güç yüksek hem de bağlı
```

### Kullanım Senaryosu — Alarm/Normal İkon Değişimi

```xml
<!-- Normal durum ikonu -->
<g id="icon_normal">
  <circle r="15" fill="#00cc00"/>
  <text text-anchor="middle" y="5" fill="white">✓</text>
</g>

<!-- Alarm durum ikonu -->
<g id="icon_alarm" display="none">
  <circle r="15" fill="#ff0000"/>
  <text text-anchor="middle" y="5" fill="white">!</text>
</g>
```

- `icon_normal` → Visibility, Expression: `ins.getVariableValue("Temperature_C").value <= 70`
- `icon_alarm` → Visibility, Expression: `ins.getVariableValue("Temperature_C").value > 70`

---

## Blink (Yanıp Sönme)

**Blink**, koşul sağlandığında SVG öğesini yanıp söndürür. Dikkat gerektiren durumlarda (aktif alarm, kritik değer, haberleşme hatası) görsel uyarı sağlar.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Blink |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Tag (Boolean), Expression |

### Çalışma Prensibi

`true` döndüğünde SVG `<animate>` elementi oluşturulur ve öğenin opacity'si 0↔1 arasında geçiş yapar. `false` döndüğünde animasyon kaldırılır.

### SVG Hazırlığı

```xml
<circle id="critical_alarm" cx="50" cy="50" r="20" fill="#ff0000"/>
```

### Expression Örnekleri

**Boolean — Tag:**
```
AlarmActive
```

**Eşik — Expression:**
```javascript
var temp = ins.getVariableValue("Temperature_C").value;
return temp > 80; // 80°C üzerinde yanıp sön
```

### Blink + Color Kombinasyonu

Aynı öğeye hem Blink hem Color uygulanabilir:

1. **Color** element: sıcaklığa göre renk (yeşil → turuncu → kırmızı)
2. **Blink** element: 80°C üzerinde yanıp sönme

Sonuç: Normal durumda yeşil sabit, 60°C'de turuncu sabit, 80°C üzerinde kırmızı yanıp söner.
