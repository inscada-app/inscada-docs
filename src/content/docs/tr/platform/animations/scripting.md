---
title: "Script & Gelişmiş"
description: "Script, FormScript, GetSymbol, Animate, Access, Three, QRCode animation tipleri"
sidebar:
  order: 5
---

Bu sayfadaki animation tipleri, gelişmiş scripting, erişim kontrolü ve özel bileşenler sağlar.

## Script

SVG öğesiyle ilişkilendirilmiş özel JavaScript kodu çalıştırır. Periyodik olarak (animation duration'ında) çalışır ve DOM'u programatik olarak günceller.

| Alan | Değer |
|------|-------|
| **Type** | Script |
| **DOM ID** | Hedef SVG öğesi |
| **Expression Type** | Expression |
| **Expression** | JavaScript kodu |

```javascript
// Birden fazla değişkenden hesaplama yapıp DOM'u güncelle
var power = ins.getVariableValue("ActivePower_kW").value;
var voltage = ins.getVariableValue("Voltage_V").value;
var current = power / voltage;

var el = document.getElementById("calculated_current");
el.textContent = current.toFixed(2) + " A";
el.setAttribute("fill", current > 50 ? "red" : "green");
```

:::tip
Basit binding'ler için Get, Color gibi hazır tipleri tercih edin. Script tipini yalnızca karmaşık mantık gerektiğinde kullanın — her animation döngüsünde çalışır ve performansı etkiler.
:::

## FormScript

Form tabanlı script bileşeni. Kullanıcıdan birden fazla girdi alarak toplu işlem yapmak için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | FormScript |
| **DOM ID** | SVG foreignObject |
| **Expression Type** | Expression |

Kullanım: Tarif değiştirme formu, toplu parametre ayarı, rapor oluşturma formu.

## GetSymbol (Sembol Yükleme)

Space seviyesindeki Symbol kütüphanesinden bir SVG sembolünü dinamik olarak yükler ve yerleştirir.

| Alan | Değer |
|------|-------|
| **Type** | GetSymbol |
| **DOM ID** | SVG group |
| **Expression** | Sembol adı |

Kullanım: Cihaz tipine göre farklı sembol gösterme — motor, pompa, vana, sensör ikonları Symbol kütüphanesinde tanımlıdır ve dinamik olarak çekilir.

## Animate (CSS/SVG Animasyon)

CSS animation veya SVG SMIL animasyonu tetikler. Dönme, titreşim, pulse gibi görsel efektler için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Animate |
| **DOM ID** | SVG öğesi |
| **Expression Type** | Tag veya Expression (Boolean) |

`value = true` → animasyon başlar, `false` → durur.

```css
/* Dönen fan animasyonu */
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

## Access (Yetki Kontrolü)

Kullanıcının yetkisine göre SVG öğesini gösterir veya gizler. Operatörlerin görmemesi gereken kontrol butonlarını, mühendislerin görmemesi gereken yönetim panellerini yetki bazlı filtrelemek için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Access |
| **DOM ID** | SVG group |
| **Expression** | Gerekli yetki adı |

Örnek: `SET_VARIABLE_VALUE` yetkisi olmayan kullanıcılar "START" butonunu göremez.

## Three (3D Görselleştirme)

Three.js tabanlı 3D görselleştirme bileşeni. SVG ekran içine 3D model gömme sağlar.

| Alan | Değer |
|------|-------|
| **Type** | Three |
| **DOM ID** | SVG foreignObject |

Kullanım: 3D tesis modeli, ekipman görselleştirme.

## QRCodeGeneration (QR Kod Oluşturma)

Değişken değerinden veya sabit metinden QR kod oluşturur ve SVG öğesine yerleştirir.

| Alan | Değer |
|------|-------|
| **Type** | QRCodeGeneration |
| **DOM ID** | SVG foreignObject veya image |
| **Expression** | QR kod içeriği |

Kullanım: Cihaz seri numarası, bakım kaydı URL'si, envanter kodu.

## QRCodeScan (QR Kod Okuma)

Kamera ile QR kod tarayıcı bileşeni. Taratılan QR koddaki veriyi değişkene veya script'e aktarır.

| Alan | Değer |
|------|-------|
| **Type** | QRCodeScan |
| **DOM ID** | SVG foreignObject |

Kullanım: Mobil cihazdan cihaz tanıma, bakım kaydı girişi, envanter takibi.
