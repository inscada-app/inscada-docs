---
title: "Pipe, Tooltip & Image"
description: "Akış animasyonu, bilgi balonu ve dinamik resim değiştirme"
sidebar:
  order: 15
---

## Pipe (Akış Animasyonu)

**Pipe**, boru hatları veya kablolarda akış yönünü gösteren animasyon oluşturur. SVG çizgi üzerinde hareket eden tire deseni ile sıvı/gaz akışını görselleştirir.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Pipe |
| **Uygun SVG Öğeleri** | `<path>`, `<line>`, `<polyline>` |
| **Expression Type** | Tag (Boolean), Expression |

### SVG Hazırlığı

```xml
<!-- Boru hattı -->
<path id="main_pipe" d="M 50,100 L 200,100 L 200,250 L 350,250"
      fill="none" stroke="#3498db" stroke-width="6"
      stroke-dasharray="15,10"/>
```

### Çalışma Prensibi

- `true` → `stroke-dashoffset` CSS animasyonu başlar, tireler hareket eder
- `false` → animasyon durur, çizgi durağan kalır

### Expression Örnekleri

**Boolean — Tag:**
```
Pump_Running
```
Pompa çalışıyorsa boru hattında akış görünür.

**Koşullu — Expression:**
```javascript
var flow = ins.getVariableValue("Flow_Rate").value;
return flow > 0; // Akış varsa animasyon
```

---

## Tooltip (Bilgi Balonu)

**Tooltip**, SVG öğesi üzerine gelindiğinde (hover) detay bilgi gösteren popup balonudur. Ana ekranda yer kaplamadan ek bilgi sağlar.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Tooltip |
| **Uygun SVG Öğeleri** | Tümü |
| **Expression Type** | Expression, Text |

### Expression Örnekleri

**Statik metin — Text:**
```
Aktif Güç Ölçümü - Trafometre çıkışı
```

**Dinamik — Expression (HTML destekler):**
```javascript
var p = ins.getVariableValue("ActivePower_kW");
var v = ins.getVariableValue("Voltage_V");
return "<b>Enerji Analizörü</b><br>"
     + "Güç: " + p.value.toFixed(1) + " kW<br>"
     + "Gerilim: " + v.value.toFixed(1) + " V<br>"
     + "Son güncelleme: " + new Date(p.dateInMs).toLocaleTimeString();
```

Kullanıcı SVG öğesi üzerine geldiğinde zengin HTML içerikli tooltip görünür.

---

## Image (Resim Değiştirme)

**Image**, bir SVG `<image>` öğesinin kaynağını (`href`) değere göre değiştirir. Durum bazlı farklı ikon, fotoğraf, sembol göstermek için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Image |
| **Uygun SVG Öğeleri** | `<image>`, `<rect>` (foreignObject ile) |
| **Expression Type** | Switch, Expression |

### SVG Hazırlığı

```xml
<image id="equipment_icon" x="50" y="50" width="64" height="64"
       href="/images/default.png"/>
```

### Switch Örneği

```
0 → /images/motor-off.png
1 → /images/motor-on.png
2 → /images/motor-fault.png
3 → /images/motor-maintenance.png
```

### Expression Örneği

```javascript
var status = ins.getVariableValue("Motor_Status").value;
var base = "/images/motor-";
if (status === 0) return base + "off.png";
if (status === 1) return base + "on.png";
if (status === 2) return base + "fault.png";
return base + "unknown.png";
```

## AlarmIndication (Alarm Göstergesi)

**AlarmIndication**, alarm grubunun durumunu otomatik olarak renk ve yanıp sönme ile gösterir. Alarm grubu tanımındaki renk ayarlarını (OnNoAck, OnAck, OffNoAck, OffAck) kullanır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | AlarmIndication |
| **Uygun SVG Öğeleri** | `<rect>`, `<circle>`, `<path>` |
| **Expression Type** | Alarm |
| **Expression** | Alarm grubu referansı |

### Otomatik Davranış

| Alarm Durumu | Görünüm |
|-------------|---------|
| Normal | Alarm grubundaki OffAck rengi |
| Fired + No Ack | OnNoAck rengi, yanıp sönme |
| Fired + Ack | OnAck rengi, sabit |
| Off + No Ack | OffNoAck rengi |

Renkleri elle ayarlamanıza gerek yoktur — alarm grubu tanımındaki renkler otomatik uygulanır.
