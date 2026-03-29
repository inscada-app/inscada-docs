---
title: "Element Editor"
description: "Animation Element Editor — SVG öğelerine değişken bağlama, ifade yazma ve canlı önizleme"
sidebar:
  order: 1
  label: "Element Editor"
---

Animation Element Editor, SVG ekranlarına interaktif davranış eklemek için kullanılan görsel düzenleyicidir. SVG içindeki öğeleri seçerek değişkenlere bağlar ve animation tipine göre gerçek zamanlı güncelleme sağlar.

## Editöre Erişim

**Menü:** Development → Animations → Animation Dev → sağ üst köşedeki **Element Editor** butonuna tıklayın.

Editör sağ panelde açılır ve SVG ekranıyla yan yana çalışır.

## İş Akışı

### 1. SVG Öğesi Seçme

**"Choose Dom Id"** butonuna tıklayarak SVG içindeki öğeleri ağaç yapısında görüntüleyin:

| Kolon | Açıklama |
|-------|----------|
| **Dom ID** | SVG öğesinin `id` özniteliği |
| **Tag Name** | Öğe tipi (text, rect, circle, g, path, image...) |
| **Attached** | Bu öğeye zaten element bağlı mı |

Listeden bir öğe seçtiğinizde, öğenin tipine göre uygun animation tipleri otomatik filtrelenir:

| SVG Öğe Tipi | Kullanılabilir Animation Tipleri |
|-------------|--------------------------------|
| **text / tspan** | Get, Color, Rotate, Move, Opacity, Visibility, Click, Tooltip, Bar, Pipe, Scale, Blink, AlarmIndication |
| **rect** | Yukarıdakilere ek: Chart, Iframe, Datatable, Slider, Input, QRCode, GetSymbol, Faceplate, Peity, Menu, Button, Image |
| **g (group)** | Animate, Faceplate, Iframe, transform animasyonları |
| **image** | Faceplate, Iframe, Image |

### 2. Animation Tipi Seçme

Filtrelenen animation tipleri sekme (tab) olarak gösterilir. İstediğiniz davranışa göre sekmeyi seçin:

- **Get** → Değer gösterimi (metin)
- **Color** → Renk değişimi
- **Rotate** → Döndürme
- **Chart** → Grafik yerleştirme
- **Set** → Tıkla-yaz kontrolü
- ... ([Tam liste →](/docs/tr/platform/animations/))

### 3. Expression Yapılandırma

Her animation element'te değerin nasıl hesaplanacağını belirleyen bir expression tanımlanır.

#### Expression Tipi Seçimi

| Tip | Açıklama | Kullanım |
|-----|----------|----------|
| **Tag** | Doğrudan değişken adı | En basit — `ActivePower_kW` yazmanız yeterli |
| **Expression** | JavaScript formülü | Karmaşık hesaplama, formatlama, koşullu mantık |
| **Switch** | Değere göre çoklu eşleşme | Durum bazlı renk/metin seçimi |
| **Numeric** | Sabit sayısal değer | Test veya sabit gösterim |
| **Text** | Sabit metin | Etiket, başlık |
| **Collection** | Birden fazla değişken | Chart, Datatable gibi çoklu veri |
| **Alarm** | Alarm durumu | AlarmIndication tipi için |
| **Faceplate** | Faceplate referansı | Faceplate yerleştirme |
| **Animation** | Başka animation referansı | Open tipi ile ekran geçişi |
| **Animation Popup** | Popup animation | Modal ekran açma |
| **Custom Menu** | Custom Menu referansı | Menu tipi ile menü açma |
| **Url** | URL adresi | Iframe tipi için |
| **Tetra Color** | Alarm 4-renk durumu | Alarm grubu renk kodları |
| **Button** | Buton yapılandırması | Button tipi için |
| **Html** | HTML içerik | Zengin içerik gömme |
| **System Page** | Sistem sayfası | Platform sayfa referansı |
| **InSCADA View** | Platform görünüm | Dahili görünüm referansı |

#### Tag Expression Örneği

En basit kullanım — değişken adını yazmanız yeterli:

```
ActivePower_kW
```

Sunucu her `duration` ms'de bu değişkenin güncel değerini okur ve SVG öğesine uygular.

#### JavaScript Expression Örnekleri

```javascript
// Formatlı değer gösterimi
var val = ins.getVariableValue("ActivePower_kW");
return val.value.toFixed(1) + " kW";
```

```javascript
// Koşullu renk
var temp = ins.getVariableValue("Temperature_C").value;
if (temp > 80) return "#ff0000";
if (temp > 60) return "#ff8800";
return "#00cc00";
```

```javascript
// Birden fazla değişkenden hesaplama
var power = ins.getVariableValue("ActivePower_kW").value;
var voltage = ins.getVariableValue("Voltage_V").value;
if (voltage > 0) return (power * 1000 / voltage).toFixed(1);
return "0";
```

#### Switch Expression Örneği

Değer → sonuç eşleşmesi. Her satır bir koşul:

```
0 → Durdu
1 → Çalışıyor
2 → Arıza
3 → Bakım
```

veya renk switch:

```
true → #00cc00
false → #ff0000
```

### 4. Props (Ek Özellikler)

Her animation tipinin kendine özel ek özellikleri vardır. Bu özellikler JSON formatında `props` alanında saklanır.

Örnekler:
- **Rotate:** döndürme merkezi (cx, cy), min/max açı
- **Bar:** yön (horizontal/vertical), min/max değer, genişlik/yükseklik
- **Slider:** min, max, step değerleri
- **Chart:** grafik tipi, renkler, eksen ayarları

### 5. Test ve Kaydetme

| Buton | İşlev |
|-------|-------|
| **Save** | Element'i veritabanına kaydeder |
| **Run & Save** | Önce expression'ı sunucuda çalıştırıp sonucu test eder, başarılıysa kaydeder |

**Run & Save** özellikle expression geliştirirken kullanışlıdır — kaydetmeden önce sonucu doğrularsınız.

---

## Çalışma Zamanı (Runtime) Mimarisi

Animation Element Editor'de yapılandırılan binding'ler çalışma zamanında şu akışla işler:

### WebSocket Tabanlı Gerçek Zamanlı Güncelleme

```
┌─────────┐                    ┌──────────┐                    ┌─────────┐
│ Tarayıcı│──eval-animation──▶│  Sunucu  │◀── Variable ──────│  Cache  │
│ (UI)    │◀─anim-results────│  (Engine) │    Values          │ (Redis) │
└────┬────┘                    └──────────┘                    └─────────┘
     │
     ▼
 DOM Güncelleme
 (SVG öğeleri)
```

### Adım Adım Akış

**1. Animation Başlatma**

Kullanıcı Visualization ekranına girdiğinde:
- Animation element'leri yüklenir
- WebSocket `animation-results` kanalına abone olunur
- İlk değerlendirme isteği gönderilir

**2. Periyodik Değerlendirme İsteği**

Her `duration` milisaniyede bir tarayıcı sunucuya WebSocket mesajı gönderir:

```json
{
  "animationId": 123,
  "callbackId": "user_session_id",
  "animName": "Energy_Dashboard",
  "sendId": "unique-request-id",
  "firstScan": false,
  "parameters": ""
}
```

**3. Sunucu Tarafı Değerlendirme**

Sunucu her element için:
1. Expression'daki değişken referanslarını çıkarır (örn: `ins.getVariableValue("ActivePower_kW")`)
2. İlgili değişken değerlerini cache'ten toplu okur
3. Her element'in expression'ını JavaScript engine'de çalıştırır
4. Sonuçları toplar

**4. Sonuç Gönderme**

```json
{
  "callbackId": "user_session_id",
  "sendId": "unique-request-id",
  "results": {
    "101": 359.91,
    "102": "#00cc00",
    "103": true,
    "104": "45.2 °C"
  },
  "isError": false
}
```

`results` map'inde key = element ID, value = expression sonucu.

**5. DOM Güncelleme**

Tarayıcıda her element için animation tipine göre uygun DOM işlemi yapılır:

| Tip | DOM İşlemi |
|-----|-----------|
| **Get** | `element.textContent = value` |
| **Color** | `element.style.fill = value` |
| **Opacity** | `element.style.opacity = value` |
| **Visibility** | `element.style.display = value ? '' : 'none'` |
| **Rotate** | `element.setAttribute('transform', 'rotate(' + value + ')')` |
| **Move** | `element.setAttribute('transform', 'translate(' + x + ',' + y + ')')` |
| **Bar** | `element.setAttribute('height', value)` |
| **Blink** | SVG animate element ekleme/kaldırma |

### Timeout ve Hata Yönetimi

- Sunucudan `duration × 10` ms içinde yanıt gelmezse → yeniden deneme
- Expression hatasında → `isError: true`, ilgili element güncellenmez
- Bağlantı koptuğunda → otomatik yeniden bağlanma

### Performans Optimizasyonu

- Tüm element'ler **tek bir WebSocket mesajında** toplu değerlendirilir
- Değişken değerleri **cache'ten** okunur (veritabanına gidilmez)
- Sadece **aktif** element'ler (`status: true`) değerlendirilir
- Click, MouseDown gibi **olay bazlı** element'ler periyodik değerlendirmeden hariçtir

---

## Kontrol (Set) Element'leri

Kontrol tipleri (Set, Slider, Input, Button, Click) farklı çalışır — periyodik değerlendirmeye dahil değildir. Kullanıcı etkileşiminde (tıklama, kaydırma vb.) tetiklenir:

```
Kullanıcı Tıklama → Set Expression Çalıştır → ins.setVariableValue() → Cache → Saha Cihazı
```

:::caution
Kontrol element'leri sunucu tarafında `ins.setVariableValue()` çağırır. Kullanıcının `SET_VARIABLE_VALUE` yetkisi olmalıdır.
:::
