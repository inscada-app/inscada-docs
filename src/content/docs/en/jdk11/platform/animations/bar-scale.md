---
title: "Bar"
description: "Değere göre çubuk yüksekliği/genişliği ayarlama"
sidebar:
  order: 12
---

**Bar**, bir SVG dikdörtgeninin yüksekliğini veya genişliğini değere orantılı olarak değiştirir. Tank seviyesi, ilerleme çubuğu, yük göstergesi, enerji barı gibi gösterimlerde kullanılır.

## Kullanım

| Alan | Değer |
|------|-------|
| **Type** | Bar |
| **Uygun SVG Öğeleri** | `<rect>` |

## Yapılandırma Tipleri

### TAG — Değişken Seçimi (Kod Yazmadan)

En hızlı kullanım. Listeden değişken seçilir, bar otomatik olarak değere göre büyür/küçülür.

![Bar — Tag](../../../../../../assets/docs/anim-bar-tag.png)

TYPE bölümünden **TAG** seçildiğinde değişken listesi ve bar yapılandırma alanları açılır.

#### Temel Alanlar

| Alan | Açıklama |
|------|----------|
| **Variable** | Açılır listeden değişken seçimi |
| **Default** | Değer okunamadığında gösterilecek varsayılan bar boyutu |

#### Bar Ayarları

| Alan | Açıklama |
|------|----------|
| **Min** | Minimum değer (bu değerde bar boş / sıfır boyut) |
| **Max** | Maksimum değer (bu değerde bar tam dolu) |
| **Direction** | Bar'ın büyüme yönü |
| **Gradient** | İşaretlenirse bar dolgusu gradient (renk geçişi) olarak gösterilir |

#### Direction (Yön) Seçenekleri

| Değer | Açıklama | Kullanım |
|-------|----------|----------|
| **Right** | Soldan sağa büyür | Yatay ilerleme çubuğu |
| **Left** | Sağdan sola büyür | Ters yönlü yatay bar |
| **Up** | Aşağıdan yukarı büyür | Tank seviyesi, dikey gösterge |
| **Down** | Yukarıdan aşağı büyür | Ters yönlü dikey bar |

### EXPRESSION — JavaScript ile Hesaplama

Birden fazla değişkenden hesaplama veya özel dönüşüm gerektiğinde kullanılır.

![Bar — Expression](../../../../../../assets/docs/anim-bar-expression.png)

TYPE bölümünden **EXPRESSION** seçildiğinde JavaScript kod editörü açılır. `return` ile döndürülen sayısal değer, Min-Max aralığına göre bar boyutuna dönüştürülür.

Expression modunda da **Min**, **Max**, **Direction** ve **Gradient** alanları aynı şekilde kullanılır.

#### Return Formatları

Expression'dan iki farklı formatta değer döndürülebilir:

**Format 1: Sayısal Değer (Basit)**

Sadece sayı döndürülür. Min/Max/Direction/Gradient ayarları editördeki form alanlarından alınır.

```javascript
// Doğrudan değişken değeri
return ins.getVariableValue('ActivePower_kW').value;
```

```javascript
// Hesaplama ile yüzde
var val = ins.getVariableValue("ActivePower_kW").value;
var maxPower = 1000;
return (val / maxPower) * 100;
```

```javascript
// İki değişkenin oranı
var output = ins.getVariableValue("Output_kW").value;
var input = ins.getVariableValue("Input_kW").value;
if (input > 0) return (output / input) * 100;
return 0;
```

**Format 2: Nesne (Dinamik Props)**

Tüm bar ayarları expression içinden dinamik olarak kontrol edilebilir. Döndürülen nesne, editördeki form alanlarını geçersiz kılar.

```javascript
var power = ins.getVariableValue("ActivePower_kW").value;
return {
    value: power,
    min: 0,
    max: 1000,
    orientation: "Right",
    fillColor: power > 800 ? "#FF0000" : "#04B3FF",
    opacity: 1,
    duration: 0.5
};
```

Nesne formatında kullanılabilir alanlar:

| Alan | Tip | Varsayılan | Açıklama |
|------|-----|-----------|----------|
| **value** | Number | — | Bar doluluk değeri (zorunlu) |
| **min** | Number | `0` | Minimum değer (bar boş) |
| **max** | Number | `100` | Maksimum değer (bar dolu) |
| **orientation** | String | `"Bottom"` | Yön: `"Bottom"`, `"Top"`, `"Left"`, `"Right"` |
| **fillColor** | String | `"#04B3FF"` | Bar dolgu rengi |
| **opacity** | Number | `1` | Saydamlık (0-1) |
| **duration** | Number | `1` | Animasyon süresi (saniye) |
| **isRadial** | Boolean | `false` | Radyal (dairesel) bar modu |
| **strokeWidth** | Number | `3` | Radyal modda çizgi kalınlığı |

:::tip
Nesne formatı ile bar rengini değere göre dinamik olarak değiştirebilirsiniz — örneğin normal aralıkta mavi, yüksek yükte kırmızı. Form ayarlarını her tarama döngüsünde programatik olarak geçersiz kılma imkanı verir.
:::

#### Radyal (Dairesel) Bar Modu

`isRadial: true` ayarı ile bar, SVG path öğesinde `stroke-dasharray` tabanlı dairesel doluluk animasyonu oluşturur. Gösterge kadranı, dairesel ilerleme çubuğu gibi gösterimlerde kullanılır.

```javascript
return {
    value: 75,
    min: 0,
    max: 100,
    isRadial: true,
    strokeWidth: 5,
    fillColor: "#00CC00",
    duration: 0.8
};
```

---

## Kullanım Örnekleri

### Yatay İlerleme Çubuğu

```xml
<svg viewBox="0 0 400 60">
  <!-- Arka plan -->
  <rect x="10" y="20" width="300" height="20" fill="#e0e0e0" rx="3"/>
  <!-- İlerleme barı -->
  <rect id="progress_bar" x="10" y="20" width="0" height="20" fill="#2196F3" rx="3"/>
</svg>
```

- TYPE: TAG, Variable: `PowerFactor`
- Min: `0`, Max: `1`, Direction: **Right**

### Dikey Tank Seviyesi

```xml
<svg viewBox="0 0 100 250">
  <!-- Tank çerçevesi -->
  <rect x="20" y="10" width="60" height="200" fill="none" stroke="#666" stroke-width="2"/>
  <!-- Seviye barı -->
  <rect id="tank_level" x="22" y="210" width="56" height="0" fill="#3498db"/>
</svg>
```

- TYPE: TAG, Variable: `Tank_Level`
- Min: `0`, Max: `100`, Direction: **Up**

:::tip
Bar animasyonu, SVG rect öğesinin `width` veya `height` özniteliğini değere orantılı olarak ayarlar. Yönüne göre `x` veya `y` koordinatı da otomatik güncellenir — rect'in sabit kalan kenarı yerinde kalır.
:::
