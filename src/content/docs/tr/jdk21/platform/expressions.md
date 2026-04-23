---
title: "Expression (Formüller)"
description: "Paylaşımlı formül tanımlama ve değişkenlerde kullanımı"
sidebar:
  order: 7
---

Expression, **space seviyesinde** tanımlanan paylaşımlı JavaScript formülleridir. Birden fazla değişken veya alarm tarafından referans olarak kullanılabilir — tekrarlayan formülleri merkezi olarak yönetmeyi sağlar.

![Expressions](../../../../../assets/docs/dev-expressions.png)

## Expression Alanları

| Alan | Tip | Zorunlu | Açıklama |
|------|-----|---------|----------|
| **name** | String (≤100) | Evet | Formül adı — space içinde benzersiz |
| **dsc** | String (≤255) | Hayır | Açıklama |
| **code** | String (≤32 767) | Evet | JavaScript kodu (ES5 — GraalJS Nashorn uyumluluk modu) |

Expression bir projeye değil, space'e bağlıdır — aynı space içindeki tüm projelerden referans verilebilir.

## Kullanım Alanları

Bir Expression iki farklı amaçla değişken tanımından referans verilebilir:

### 1. Value Expression (değer üretme)

Değişkenin değerini hesaplamak için her okuma döngüsünde çalışır. Variable'ın `valueExpressionType` alanı (`ExpressionType` enum) üç değer alır:

| Tip | Açıklama |
|-----|----------|
| **NONE** | Expression yok — ham (ölçeklenmiş) değer kullanılır |
| **CUSTOM** | Değişkene özel, inline JavaScript (`valueExpressionCode`) |
| **EXPRESSION** | Paylaşımlı Expression'a referans (`valueExpressionId`) |

`EXPRESSION` seçildiğinde, değişken tanımında Expression kimliği belirtilir. Bu sayede aynı formül onlarca değişkende kullanılabilir; formül değiştiğinde tüm referans veren değişkenler otomatik etkilenir.

### 2. Log Expression (loglama kararı)

Değişkenin ne zaman loglanacağını belirleyen özel koşul — `logType = Expression` veya `logType = Custom` ayarlandığında kullanılır. Truthy dönerse loglanır, falsy dönerse atlanır.

```javascript
// Sadece değer belirli aralıkta ise logla
if (value > 100 && value < 900) {
    return true;
}
return false;
```

## Örnek Expression'lar

### Birim Dönüşümü

```javascript
// Fahrenheit → Celsius (birden fazla sıcaklık sensöründe kullanılır)
return ((value - 32) * 5 / 9).toFixed(1) * 1;
```

### Ölçek Normalizasyonu

```javascript
// 0-65535 raw değeri 0-100 yüzdeye çevir
return (value / 65535 * 100).toFixed(1) * 1;
```

### Durum Metni

```javascript
// Sayısal durum kodunu metne çevir
var states = { 0: "Durdu", 1: "Çalışıyor", 2: "Arıza", 3: "Bakım" };
return states[value] || "Bilinmiyor";
```

### Birden Fazla Değişkene Bağlı Hesaplama

```javascript
// Anlık verim yüzdesi
var input = ins.getVariableValue("Input_kW").value;
var output = ins.getVariableValue("Output_kW").value;
if (input > 0) {
    return ((output / input) * 100).toFixed(1) * 1;
}
return 0;
```

## Expression Ortamı

Expression'lar Variable / Alarm motorunun içinde GraalJS üzerinde çalışır:
- `value` — değişkenin ham (ölçekten önceki) değeri (value expression'larda bir önceki hesaplanan değer)
- `ins.*` — tüm server-side API'ye erişim (başka değişkenleri okumak için `ins.getVariableValue()` vb.)
- ES5 söz dizimi önerilir (Nashorn uyumluluğu için) — `var`, `function`, `for`, `if/else`, `try/catch`. `let`, `const`, arrow (`=>`), template string ve `class` çalışır ancak JDK11 ile uyumluluk korunmak isteniyorsa ES5'te kalın.

## Space Seviyesi Avantajı

- Bir formülü değiştirdiğinizde, onu `EXPRESSION` olarak referans veren **tüm değişkenler** aynı anda güncel formülü kullanır
- Farklı projelerdeki değişkenler aynı formülü paylaşabilir
- Sık kullanılan dönüşümler (birim, ölçek, durum kodu) için bir "formül kütüphanesi" oluşturulabilir
