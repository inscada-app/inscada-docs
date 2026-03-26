---
title: "DNP3"
description: "inSCADA'da DNP3 Master ve Slave (Outstation) bağlantı yapılandırması"
sidebar:
  order: 2
---

DNP3 (Distributed Network Protocol), elektrik dağıtım, su/atıksu, ulaşım ve petrol/gaz sektörlerinde kontrol merkezi bilgisayarları, RTU'lar ve IED'ler (Akıllı Elektronik Cihazlar) arasında standart tabanlı haberleşme sağlamak için geliştirilmiş bir protokoldür.

inSCADA, DNP3 protokolünü hem **Master** hem de **Slave (Outstation)** rolünde destekler. TCP/IP üzerinden çalışır.

## Temel Kavramlar

### Master ve Outstation

DNP3 haberleşmesinde iki temel rol vardır:

- **Master:** Outstation'lara sorgu göndererek veri toplayan taraf. inSCADA Master rolünde saha cihazlarından veri okur ve kontrol komutları gönderir.
- **Outstation (Slave):** Veritabanında saha verilerini tutan ve Master'ın sorgularına yanıt veren taraf. inSCADA Outstation rolünde dış sistemlere veri sunar.

### Veri Tipleri (Object Groups)

DNP3, veri tiplerini **Group** ve **Variation** numaralarıyla sınıflandırır:

| Veri Tipi | Açıklama | Okuma | Yazma |
|-----------|----------|:-----:|:-----:|
| **Binary Input** | Dijital giriş (açık/kapalı) | ✓ | — |
| **Double Input** | Çift-bit dijital giriş | ✓ | — |
| **Binary Output** | Dijital çıkış (kontrol) | ✓ | ✓ |
| **Counter** | Sayaç değeri (kWh vb.) | ✓ | — |
| **Frozen Counter** | Dondurulmuş sayaç | ✓ | — |
| **Analog Input** | Analog ölçüm değeri | ✓ | — |
| **Analog Output** | Analog çıkış (setpoint) | ✓ | ✓ |

### Static ve Event Veriler

- **Static Data:** Anlık değerler — binary input'un o anki durumu, analog değerin mevcut ölçümü
- **Event Data:** Durum değişiklikleri, eşik aşımları gibi olaylar. Zaman damgası ile veya zaman damgasız raporlanabilir.

DNP3 olayları üç sınıfa ayırır:
- **Class 1:** En yüksek öncelik
- **Class 2:** Orta öncelik
- **Class 3:** Düşük öncelik

### Unsolicited Response

Outstation'lar, Master'ın sorgusu olmadan değişiklik bildirimi gönderebilir. Bu özellik, çok sayıda outstation bulunan sistemlerde polling döngüsünü beklemeden anlık bildirim sağlar.

## Veri Modeli

```
Connection (Bağlantı)
└── Device (Cihaz — Local/Remote adres çifti)
    └── Frame (Veri Bloğu — Object group tanımı)
        └── Variable (Değişken — Point index)
```

### Connection Parametreleri

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **IP / Host** | String | Hedef cihaz IP adresi |
| **Port** | Integer | Hedef port (varsayılan: 20000) |
| **Adapter** | String | Ağ adaptörü seçimi |
| **Pool Size** | Integer | Bağlantı havuzu boyutu |
| **Min Retry Delay** | Integer (ms) | Minimum yeniden bağlanma bekleme süresi |
| **Max Retry Delay** | Integer (ms) | Maksimum yeniden bağlanma bekleme süresi |

### Device Parametreleri

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **Local Address** | Integer | Master'ın DNP3 adresi (genellikle 1) |
| **Remote Address** | Integer | Outstation'ın DNP3 adresi |
| **Response Timeout** | Integer (ms) | Yanıt bekleme süresi |
| **Integrity Scan Time** | Integer (ms) | Bütünlük tarama periyodu (tüm static veriler) |
| **Event Scan Time** | Integer (ms) | Olay tarama periyodu (Class 1/2/3 eventler) |
| **Scan Type** | Enum | `PERIODIC` veya `FIXED_DELAY` |
| **Unsolicited Events** | Boolean | Unsolicited response'ları kabul et |
| **Disable Unsolicited on Startup** | Boolean | Başlangıçta unsolicited'ı devre dışı bırak |
| **Startup Integrity** | Boolean | Başlangıçta bütünlük taraması yap |
| **Integrity on Event Overflow IIN** | Boolean | Event buffer taşmasında bütünlük taraması |

:::tip
**Integrity Scan** tüm static verileri sorgular ve outstation veritabanının tam bir kopyasını alır. **Event Scan** yalnızca son taramadan bu yana değişen verileri getirir. Her iki tarama da ayrı periyotlarla yapılandırılabilir.
:::

### Frame (Veri Bloğu) Parametreleri

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **Type** | Enum | Veri tipi (Binary Input, Analog Input, Counter vb.) |
| **Start Address** | Integer | Başlangıç point index |
| **Quantity** | Integer | Point sayısı |
| **Event Buffer Size** | Integer | Olay tamponu boyutu |
| **Point Class** | String | Olay sınıfı (Class 1, 2 veya 3) |
| **Static Variation** | String | Static veri formatı (ör: Group30Var5 — 32-bit float) |
| **Event Variation** | String | Event veri formatı |
| **Deadband** | Double | Analog olay eşiği (değişim miktarı) |

### Variable Parametreleri

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| **Start Address** | Integer | Point index |
| **Point Class** | String | Olay sınıfı |
| **Static Variation** | String | Static veri formatı |
| **Event Variation** | String | Event veri formatı |
| **Deadband** | Double | Analog olay eşiği |

## DNP3 Master Yapılandırma

### Adım 1: Connection Oluşturma

Protokol olarak **DNP3** seçin ve hedef outstation'ın IP ve port bilgilerini girin.

### Adım 2: Device Oluşturma

- **Local Address:** Master adresi (genellikle 1)
- **Remote Address:** Outstation adresi (cihaz dokümantasyonundan)
- **Integrity Scan Time:** Önerilen başlangıç değeri 60000 ms (1 dakika)
- **Event Scan Time:** Önerilen başlangıç değeri 5000 ms (5 saniye)

### Adım 3: Frame ve Variable Tanımlama

Outstation'daki her veri grubu için bir Frame oluşturun. Örnek:

- **Analog Input Frame:** Type = `Analog Input`, Start = 0, Quantity = 10
- **Binary Input Frame:** Type = `Binary Input`, Start = 0, Quantity = 32

Frame içindeki her point index için Variable oluşturun.

### Adım 4: Bağlantıyı Başlatma

**Runtime Control Panel**'den bağlantıyı başlatın. Bağlantı durumu "Connected" olarak görünecektir.

## DNP3 Slave (Outstation) Yapılandırma

inSCADA, DNP3 Outstation rolünde çalışarak dış Master'ların bağlanıp veri okumasına izin verir. Bu mod genellikle:

- Üst SCADA sistemlerine veri sunmak
- Kontrol merkezine DNP3 üzerinden veri aktarmak
- Farklı inSCADA instance'ları arasında veri paylaşımı

için kullanılır.

Yapılandırma için protokol olarak **DNP3 Slave** seçin. Device'da Local Address ve Remote Address rolleri tersine döner — Local Address outstation'ın kendi adresi, Remote Address bağlanacak Master'ın adresidir.

:::caution
DNP3 Slave portu, firewall'da **gelen bağlantılara açık** olmalıdır.
:::
