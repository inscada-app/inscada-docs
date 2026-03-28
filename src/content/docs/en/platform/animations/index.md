---
title: "SVG Animations"
description: "SVG tabanlı interaktif SCADA ekranları — genel bakış, tasarım ve yapı"
sidebar:
  order: 0
  label: "Genel Bakış"
---

SVG Animation, inSCADA'nın temel görselleştirme bileşenidir. Her animation bir SVG dosyasından oluşur ve değişken değerlerine bağlanarak gerçek zamanlı SCADA ekranları oluşturur.

![Energy Monitoring Dashboard](../../../../../assets/docs/variable-tracking.png)

## Animation Dev Ekranı

**Menü:** Development → Animations → Animation Dev

![Animation Dev](../../../../../assets/docs/dev-animation-dev.png)

## Animation Oluşturma

| Alan | Zorunlu | Açıklama |
|------|---------|----------|
| **Name** | Evet | Ekran adı (proje içinde benzersiz) |
| **SVG Content** | Evet | SVG kaynak kodu |
| **Duration** | Evet | Animasyon güncelleme periyodu (ms, min: 100) |
| **Play Order** | Evet | Sıralama (birden fazla ekran varsa) |
| **Main** | Evet | Ana ekran mı |
| **Color** | Hayır | Arka plan rengi |
| **Description** | Hayır | Açıklama |

## Animation Yapısı

Her animation üç bileşenden oluşur:

```
Animation
├── SVG Content (ekranın görsel yapısı)
├── Animation Elements (değişken bağlantıları)
│   ├── Element 1: "temp_text" → Temperature_C (Get binding)
│   ├── Element 2: "motor_rect" → MotorStatus (Color binding)
│   └── Element 3: "valve_group" → ValvePosition (Rotate binding)
└── Animation Scripts (ekran scriptleri)
    ├── Pre-Animation Code
    └── Post-Animation Code
```

## Animation Elements

Animation Element, SVG içindeki bir DOM öğesini bir değişkene bağlar. Her element şu alanlardan oluşur:

| Alan | Zorunlu | Açıklama |
|------|---------|----------|
| **Name** | Evet | Element adı |
| **DOM ID** | Evet | SVG içindeki hedef öğenin `id` özniteliği |
| **Type** | Evet | Animation tipi (binding davranışı) |
| **Expression Type** | Evet | İfade tipi (değerin nasıl hesaplanacağı) |
| **Expression** | Evet | Değer ifadesi (değişken adı, formül vb.) |
| **Props** | Evet | Ek özellikler (JSON) |
| **Status** | Evet | Aktif/pasif |

### Animation Tipleri

inSCADA **36 farklı animation tipi** destekler:

#### Veri Gösterimi

| Tip | Açıklama | Sayfa |
|-----|----------|-------|
| **Get** | Değişken değerini metin olarak göster | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Color** | Öğenin rengini değere göre değiştir | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Bar** | Değere göre çubuk yüksekliği/genişliği | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Opacity** | Değere göre saydamlık | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Visibility** | Koşula göre göster/gizle | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Rotate** | Değere göre döndürme | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Move** | Değere göre X/Y kaydırma | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Scale** | Değere göre ölçekleme | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Blink** | Koşula göre yanıp sönme | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Pipe** | Boru/hat akış animasyonu | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Tooltip** | Hover bilgi balonu | [Detay →](/docs/tr/platform/animations/data-display/) |
| **Image** | Değere göre resim değiştirme | [Detay →](/docs/tr/platform/animations/data-display/) |
| **AlarmIndication** | Alarm durumunu göster | [Detay →](/docs/tr/platform/animations/data-display/) |

#### Grafik & Veri Tablosu

| Tip | Açıklama | Sayfa |
|-----|----------|-------|
| **Chart** | Grafik bileşeni | [Detay →](/docs/tr/platform/animations/charts-tables/) |
| **Peity** | Inline sparkline mini grafik | [Detay →](/docs/tr/platform/animations/charts-tables/) |
| **Datatable** | Tablo bileşeni | [Detay →](/docs/tr/platform/animations/charts-tables/) |

#### Kontrol & Etkileşim

| Tip | Açıklama | Sayfa |
|-----|----------|-------|
| **Set** | Değişkene değer yaz (click ile) | [Detay →](/docs/tr/platform/animations/controls/) |
| **Slider** | Kaydırıcı ile değer ayarla | [Detay →](/docs/tr/platform/animations/controls/) |
| **Input** | Metin/sayı girişi | [Detay →](/docs/tr/platform/animations/controls/) |
| **Button** | Buton bileşeni | [Detay →](/docs/tr/platform/animations/controls/) |
| **Click** | Tıklama olayı | [Detay →](/docs/tr/platform/animations/controls/) |
| **MouseDown / MouseUp / MouseOver** | Fare olayları | [Detay →](/docs/tr/platform/animations/controls/) |

#### Navigasyon & Gömme

| Tip | Açıklama | Sayfa |
|-----|----------|-------|
| **Open** | Başka bir animation'a geç | [Detay →](/docs/tr/platform/animations/navigation/) |
| **Iframe** | Harici URL gömme | [Detay →](/docs/tr/platform/animations/navigation/) |
| **Menu** | Menü açma | [Detay →](/docs/tr/platform/animations/navigation/) |
| **Faceplate** | Faceplate bileşeni yerleştir | [Detay →](/docs/tr/platform/animations/navigation/) |

#### Script & Gelişmiş

| Tip | Açıklama | Sayfa |
|-----|----------|-------|
| **Script** | Özel JavaScript çalıştır | [Detay →](/docs/tr/platform/animations/scripting/) |
| **FormScript** | Form tabanlı script | [Detay →](/docs/tr/platform/animations/scripting/) |
| **GetSymbol** | Symbol kütüphanesinden sembol yükle | [Detay →](/docs/tr/platform/animations/scripting/) |
| **Animate** | CSS/SVG animasyon tetikle | [Detay →](/docs/tr/platform/animations/scripting/) |
| **Access** | Yetki bazlı görünürlük | [Detay →](/docs/tr/platform/animations/scripting/) |
| **Three** | 3D görselleştirme | [Detay →](/docs/tr/platform/animations/scripting/) |
| **QRCodeGeneration** | QR kod oluşturma | [Detay →](/docs/tr/platform/animations/scripting/) |
| **QRCodeScan** | QR kod okuma | [Detay →](/docs/tr/platform/animations/scripting/) |

### Expression Tipleri

Her animation element'te değerin nasıl hesaplanacağını belirler:

| Tip | Açıklama |
|-----|----------|
| **Tag** | Doğrudan değişken adı referansı |
| **Expression** | JavaScript formülü |
| **Numeric** | Sabit sayısal değer |
| **Text** | Sabit metin değeri |
| **Switch** | Değere göre koşullu seçim |
| **Collection** | Birden fazla değişken koleksiyonu |
| **Set** | Değer yazma ifadesi |
| **Animation** | Başka bir animation referansı |
| **Url** | URL referansı |
| **Alarm** | Alarm durumu referansı |
| **Faceplate** | Faceplate referansı |
| **Animation Popup** | Popup animation açma |
| **Button** | Buton yapılandırması |
| **InSCADA View** | Platform görünüm referansı |
| **System Page** | Sistem sayfası referansı |
| **Html** | HTML içerik |
| **Custom Menu** | Custom menu referansı |
| **Tetra Color** | Dört renkli durum gösterimi (alarm renkleri) |

## Animation Scripts

Her animation'a script bağlanabilir:

| Script | Çalışma Zamanı | Kullanım |
|--------|---------------|----------|
| **Pre-Animation Code** | Ekran açıldığında | Başlangıç değerleri, veri çekme |
| **Post-Animation Code** | Ekran kapandığında | Temizlik |

## SVG Tasarım İlkeleri

### ID Kuralları

SVG öğelerine anlamlı `id` değerleri verin — Animation Element `domId` alanı bunları referans eder:

```xml
<text id="temp_display">--</text>
<rect id="motor_indicator" fill="#cccccc"/>
<g id="valve_group" transform="rotate(0)">
  <path d="..."/>
</g>
```

### Gerçek Zamanlı Güncelleme

Animation açıldığında WebSocket bağlantısı kurulur. Platform, `duration` parametresinde belirtilen aralıkta değişken değerlerini istemciye push eder ve binding'ler otomatik güncellenir.

### Placeholder (Parametrik Ekran)

Animation'lara placeholder tanımlanabilir. Aynı SVG tasarımı farklı parametrelerle (farklı cihaz, farklı değişken seti) tekrar kullanılabilir.
