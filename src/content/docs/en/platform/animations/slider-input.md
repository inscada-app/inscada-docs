---
title: "Slider & Input"
description: "Kaydırıcı ve metin/sayı giriş kontrolleri"
sidebar:
  order: 18
---

## Slider (Kaydırıcı)

**Slider**, sürüklenebilir kaydırıcı ile analog değer ayarlamak için kullanılır. Setpoint, hız ayarı, sıcaklık hedefi, dimmer gibi sürekli değer kontrolleri.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Slider |
| **Uygun SVG Öğeleri** | `<rect>` (foreignObject) |
| **Expression Type** | Tag, Expression |

### Yapılandırma (Props)

| Özellik | Açıklama | Örnek |
|---------|----------|-------|
| **min** | Minimum değer | 0 |
| **max** | Maksimum değer | 100 |
| **step** | Adım büyüklüğü | 1 |
| **title** | Başlık | "Sıcaklık Setpoint" |

### SVG Hazırlığı

```xml
<foreignObject id="temp_slider" x="50" y="200" width="300" height="60"/>
```

### Yapılandırma

- Expression Type: **Tag** → `Temperature_Setpoint`
- Props: min=0, max=100, step=0.5

Operatör kaydırıcıyı sürüklediğinde değer anında değişkene yazılır.

### Expression ile İleri Düzey

```javascript
// Değer yazıldığında loglama
var newValue = value; // slider'dan gelen değer
ins.setVariableValue("Temperature_Setpoint", {value: newValue});
ins.writeLog("INFO", "Setpoint", "Temperature setpoint: " + newValue + "°C");
return newValue;
```

---

## Input (Giriş Alanı)

**Input**, metin veya sayı giriş alanı oluşturur. Operatör klavyeden değer girer ve Enter ile onaylar.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Input |
| **Uygun SVG Öğeleri** | `<rect>` (foreignObject) |
| **Expression Type** | Tag, Expression |

### Yapılandırma (Props)

| Özellik | Açıklama | Örnek |
|---------|----------|-------|
| **type** | Giriş tipi | `number`, `text` |
| **min** | Minimum değer (number) | 0 |
| **max** | Maksimum değer (number) | 100 |
| **placeholder** | Boş iken gösterilen metin | "Setpoint girin..." |

### SVG Hazırlığı

```xml
<foreignObject id="setpoint_input" x="100" y="150" width="150" height="35"/>
```

### Kullanım Senaryoları

**Sayısal setpoint:**
- Expression Type: Tag → `Temperature_Setpoint`
- Props: type=number, min=0, max=100

**Tarif adı:**
- Expression Type: Tag → `Recipe_Name`
- Props: type=text, placeholder="Tarif adı..."

**Parametre girişi:**
- Expression Type: Expression
```javascript
var val = parseFloat(value);
if (val < 0 || val > 100) {
    ins.notify("error", "Hata", "Değer 0-100 aralığında olmalı!");
    return;
}
ins.setVariableValue("Setpoint", {value: val});
ins.notify("success", "OK", "Setpoint güncellendi: " + val);
```
