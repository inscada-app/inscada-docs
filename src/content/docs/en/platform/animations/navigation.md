---
title: "Navigasyon & Gömme"
description: "Open, Iframe, Menu ve Faceplate animation tipleri"
sidebar:
  order: 4
---

Bu sayfadaki animation tipleri, ekranlar arası geçiş ve harici içerik gömme sağlar.

## Open (Ekran Geçişi)

Tıklandığında başka bir animation ekranına geçiş yapar. SCADA ekranları arasında navigasyon için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Open |
| **DOM ID** | Tıklanabilir SVG öğesi |
| **Expression Type** | Animation |
| **Expression** | Hedef animation adı |

```xml
<!-- Detay ekranına geçiş butonu -->
<g id="goto_motor_detail" cursor="pointer">
  <rect width="100" height="30" rx="5" fill="#1CA1C1"/>
  <text x="50" y="20" text-anchor="middle" fill="white">Motor Detay →</text>
</g>
```

Tıklandığında "Motor_Detail" animation'ı açılır.

### Animation Popup

`Animation Popup` expression tipi ile hedef ekranı popup (modal pencere) olarak açabilirsiniz. Ana ekran arka planda kalır.

## Iframe (Harici URL Gömme)

SVG ekran içine harici web sayfası gömer. Grafana dashboard, kamera görüntüsü, harici web uygulaması gibi içerikleri SCADA ekranına entegre eder.

| Alan | Değer |
|------|-------|
| **Type** | Iframe |
| **DOM ID** | SVG foreignObject |
| **Expression Type** | Url |
| **Expression** | Hedef URL |

```xml
<foreignObject id="grafana_panel" x="10" y="300" width="600" height="400"/>
```

Expression: `https://grafana.company.com/d/energy/panel?orgId=1&kiosk`

## Menu (Menü Açma)

Tıklandığında Custom Menu sayfasını açar.

| Alan | Değer |
|------|-------|
| **Type** | Menu |
| **DOM ID** | Tıklanabilir SVG öğesi |
| **Expression Type** | Custom Menu |
| **Expression** | Custom Menu adı |

## Faceplate (Bileşen Yerleştirme)

SVG ekran içine bir Faceplate bileşenini yerleştirir. Tekrar kullanılabilir motor, vana, pompa sembollerini animation ekranına gömmek için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Faceplate |
| **DOM ID** | SVG group |
| **Expression Type** | Faceplate |
| **Expression** | Faceplate adı + placeholder değerleri |

```xml
<!-- Motor 1 faceplate alanı -->
<g id="motor1_faceplate" transform="translate(100, 200)"/>

<!-- Motor 2 faceplate alanı -->
<g id="motor2_faceplate" transform="translate(300, 200)"/>
```

Her iki alan da aynı "Motor_Standard" faceplate'ini kullanır, farklı placeholder değerleri ile:

- `motor1_faceplate`: `{motor_name}=Motor 1, {speed_var}=M1_Speed`
- `motor2_faceplate`: `{motor_name}=Motor 2, {speed_var}=M2_Speed`

Detaylı bilgi: [Faceplate →](/docs/tr/platform/faceplates/)
