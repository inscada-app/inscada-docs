---
title: "Open, Iframe & Faceplate"
description: "Ekran geçişi, harici URL gömme ve faceplate yerleştirme"
sidebar:
  order: 19
---

## Open (Ekran Geçişi)

**Open**, tıklandığında başka bir animation ekranına geçiş yapar. SCADA ekranları arasında hiyerarşik navigasyon kurmak için kullanılır.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Open |
| **Uygun SVG Öğeleri** | Tümü (tıklanabilir) |
| **Expression Type** | Animation, Animation Popup |

### SVG Hazırlığı

```xml
<g id="goto_motor_detail" cursor="pointer">
  <rect width="120" height="35" rx="5" fill="#1CA1C1"/>
  <text x="60" y="22" text-anchor="middle" fill="white" font-size="13">
    Motor Detay →
  </text>
</g>
```

### Expression Tipleri

| Expression Type | Davranış |
|----------------|---------|
| **Animation** | Mevcut ekranı kapatıp hedef animation'ı açar |
| **Animation Popup** | Hedef animation'ı modal popup olarak açar (ana ekran arka planda kalır) |

### Navigasyon Hiyerarşisi Örneği

```
Ana Ekran (Genel Bakış)
├── [Open] → Motor Bölümü
│   ├── [Open] → Motor 1 Detay
│   └── [Open] → Motor 2 Detay
├── [Open] → Pompa Bölümü
└── [Open Popup] → Alarm Özet (modal)
```

### Parametre Geçişi

Open ile açılan animation'a placeholder parametreleri geçirilebilir:

```
Hedef: Motor_Detail
Parametreler: motor_id=1, motor_name=Motor 1
```

Hedef ekrandaki `{motor_id}` ve `{motor_name}` placeholder'ları bu değerlerle doldurulur.

---

## Iframe (Harici URL Gömme)

**Iframe**, SVG ekran içine harici web sayfası gömer. Grafana dashboard, IP kamera, harici web uygulaması, PDF doküman gibi içerikleri SCADA ekranına entegre eder.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Iframe |
| **Uygun SVG Öğeleri** | `<rect>` (foreignObject), `<g>` |
| **Expression Type** | Url |

### SVG Hazırlığı

```xml
<rect id="grafana_embed" x="10" y="300" width="600" height="400"
      fill="#f5f5f5" stroke="#ddd"/>
```

### Expression Örnekleri

**Statik URL:**
```
https://grafana.company.com/d/energy/panel?orgId=1&kiosk
```

**IP Kamera:**
```
http://192.168.1.50/video.cgi
```

**Dinamik URL — Expression:**
```javascript
var projectId = 153;
return "https://grafana.company.com/d/energy?var-project=" + projectId + "&kiosk";
```

### Kullanım Senaryoları

| İçerik | URL Formatı |
|--------|-------------|
| Grafana Dashboard | `https://grafana/d/xxx?kiosk` |
| IP Kamera MJPEG | `http://camera-ip/video` |
| PDF Rapor | `/files/reports/daily.pdf` |
| Custom Menu HTML | Platform dahili Custom Menu sayfası |
| Harici Web App | Herhangi bir web uygulaması |

---

## Faceplate (Bileşen Yerleştirme)

**Faceplate**, önceden tasarlanmış tekrar kullanılabilir SVG bileşenini animation ekranına yerleştirir. Aynı sembolü farklı parametrelerle birden fazla kez kullanmayı sağlar.

### Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Faceplate |
| **Uygun SVG Öğeleri** | `<g>`, `<rect>`, `<image>` |
| **Expression Type** | Faceplate |

### SVG Hazırlığı

```xml
<!-- Motor 1 alanı -->
<g id="motor1_area" transform="translate(100, 200)"/>

<!-- Motor 2 alanı -->
<g id="motor2_area" transform="translate(350, 200)"/>

<!-- Motor 3 alanı -->
<g id="motor3_area" transform="translate(600, 200)"/>
```

### Yapılandırma

Her alan için aynı Faceplate, farklı placeholder değerleri:

**motor1_area:**
- Faceplate: `Motor_Standard`
- Placeholders: `{motor_name}=Motor 1, {speed_var}=M1_Speed, {status_var}=M1_Status`

**motor2_area:**
- Faceplate: `Motor_Standard`
- Placeholders: `{motor_name}=Motor 2, {speed_var}=M2_Speed, {status_var}=M2_Status`

**motor3_area:**
- Faceplate: `Motor_Standard`
- Placeholders: `{motor_name}=Motor 3, {speed_var}=M3_Speed, {status_var}=M3_Status`

3 motor, 1 faceplate tasarımı, 3 placeholder seti.

### Menu (Menü Açma)

**Menu** tipi, tıklandığında Custom Menu sayfasını açar.

| Alan | Değer |
|------|-------|
| **Type** | Menu |
| **Expression Type** | Custom Menu |
| **Expression** | Custom Menu adı |

Detaylı bilgi: [Faceplate →](/docs/tr/platform/faceplates/) | [Custom Menus →](/docs/tr/platform/custom-menus/)
