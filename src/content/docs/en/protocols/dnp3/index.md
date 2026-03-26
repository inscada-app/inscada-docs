---
title: "DNP3"
description: "inSCADA'da DNP3 protokolü — genel bakış, mimari ve ortak parametreler"
sidebar:
  order: 0
  label: "DNP3"
---

DNP3 (Distributed Network Protocol), elektrik dağıtım sektöründe kontrol merkezi bilgisayarları, RTU'lar ve IED'ler (Akıllı Elektronik Cihazlar) arasında standart tabanlı haberleşme sağlamak amacıyla geliştirilmiştir. Günümüzde elektrik enerjisi dışında su/atıksu, ulaşım ve petrol/gaz sektörlerinde de yaygın olarak kullanılmaktadır.

DNP3, IEC Teknik Komite 57 standartlarına uygun olarak OSI 3 katmanlı "Enhanced Performance Architecture" (EPA) protokolü olarak çalışır. İlk olarak Kasım 1993'te Harris Corporation ve Distributed Automation Products tarafından geliştirilmiş, sahipliği daha sonra DNP3 User Group organizasyonuna devredilmiştir.

inSCADA, DNP3 protokolünü hem **Master** hem de **Outstation (Slave)** rolünde, TCP/IP üzerinden destekler.

## Protokol Amacı

DNP3, özellikle SCADA uygulamaları için — veri toplama ve kontrol komutlarının bilgisayarlar arasında iletimi amacıyla optimize edilmiştir. Genel amaçlı internet protokollerinden (e-posta, doküman, multimedya) farklı olarak endüstriyel haberleşmeye odaklanır.

## Master ve Outstation

DNP3 haberleşmesinde iki temel rol vardır:

- **Master:** Outstation'lara sorgu (polling) göndererek veri toplayan, kontrol komutları ileten taraf
- **Outstation (Slave):** Veritabanında saha verilerini tutan, Master'ın sorgularına yanıt veren taraf

Master, Outstation veritabanlarını periyodik polling ile günceller — Outstation'lardan veritabanı değerlerini göndermelerini talep eder.

## Sistem Topolojileri

**Bire-bir:** Tek Master, tek Outstation ile özel hat veya telefon hattı üzerinden haberleşir.

**Multi-drop:** Tek Master, birden fazla Outstation ile sıralı olarak haberleşir. Her Outstation yalnızca kendisine adreslenmiş mesajlara yanıt verir. Haberleşme çoklu telefon hattı, fiber optik veya radyo link üzerinden gerçekleşir.

**Hiyerarşik:** Orta birim hem Master (alt Outstation'lar için) hem de Outstation (üst Master için) rolünü üstlenir — Sub-Master olarak adlandırılır.

**TCP/IP:** Birçok üretici, geleneksel haberleşme katmanı yerine TCP/IP üzerinden DNP3 mesajları taşıyan ürünler sunmaktadır. Bu sayede coğrafi olarak dağınık cihazlardan ekonomik veri toplama mümkündür.

## Veri Tipleri (Object Groups)

DNP3, verileri **Group** ve **Variation** numaralarıyla sınıflandırır:

| Veri Tipi | Açıklama | Okuma | Yazma |
|-----------|----------|:-----:|:-----:|
| **Binary Input** | Dijital giriş — fiziksel veya lojik boolean durum | ✓ | — |
| **Double Input** | Çift-bit dijital giriş | ✓ | — |
| **Binary Output** | Dijital çıkış — açma/kapama, çalıştırma/durdurma | ✓ | ✓ |
| **Counter** | Sayaç değeri (kWh vb.) | ✓ | — |
| **Frozen Counter** | Dondurulmuş sayaç | ✓ | — |
| **Analog Input** | Ölçülen veya hesaplanan analog değer | ✓ | — |
| **Analog Output** | Analog çıkış — setpoint gibi fiziksel/lojik değer | ✓ | ✓ |

### Static ve Event Veriler

**Static Data:** Anlık değerleri temsil eder — binary input'un o anki açık/kapalı durumu veya iletim anındaki analog değer.

**Event Data:** Durum değişiklikleri, eşik aşımları, anlık veri gibi önemli olaylarla ilişkilendirilir. DNP3, olayları **zaman damgası ile veya zaman damgasız** raporlama imkânı sunar.

Outstation'lar, binary input durum değişikliği veya analog değerin yapılandırılmış **deadband** limitini aşması durumunda olay üretir.

### Olay Sınıfları

DNP3 olayları üç sınıfa ayırır:
- **Class 1:** En yüksek öncelik
- **Class 2:** Orta öncelik
- **Class 3:** Düşük öncelik

Master, Class 1, 2, 3 veya kombinasyonlarını ayrı ayrı sorgulayabilir.

### Analog Variation'lar

Static analog veri formatları:
1. 32-bit integer (flag'li)
2. 16-bit integer (flag'li)
3. 32-bit integer
4. 16-bit integer
5. **32-bit floating point (flag'li)**
6. 64-bit floating point (flag'li)

**Flag:** Online durumu, restart, haberleşme kaybı, zorlanmış veri ve aralık dışı veri bilgilerini içeren 8-bit bayrak.

### Unsolicited Response

Outstation'lar, Master'ın sorgusu olmadan yanıt gönderebilir. Bu mod, çok sayıda Outstation bulunan sistemlerde Master'ın değişikliklerden anında haberdar olması gerektiğinde kullanışlıdır. Polling döngüsünü beklemek yerine Outstation değişiklikleri doğrudan iletir.

## Katman Mimarisi

### Link Katmanı
Güvenilir fiziksel bağlantıyı hata tespiti ve çift frame tespiti ile sağlar. Maksimum payload 250 byte (CRC hariç), toplam maksimum frame uzunluğu CRC ve header'lar dahil **292 byte**'tır.

- **CRC:** Her 16 veri byte'ı için bir CRC oktet çifti ile haberleşme hatası tespiti
- **Adresleme:** 65520 ayrı adres mevcuttur

### Transport Katmanı
Uzun uygulama katmanı mesajlarını link katmanı iletimi için küçük paketlere böler. Yalnızca 1 byte overhead gerektirir.

### Uygulama Katmanı (Fragment)
Mesajlar, alıcı cihazın buffer boyutuna göre fragment'lara bölünür (tipik: 2048-4096 byte). Gürültülü ortamlarda fragment boyutunun küçültülmesi haberleşme başarısını artırabilir.

## Implementation Level'ları

DNP3, tüm cihazların tüm özelliklere ihtiyaç duymadığını kabul eder:

- **Level 1:** Yalnızca temel fonksiyonlar
- **Level 2:** Daha fazla fonksiyon, group ve variation
- **Level 3:** En kapsamlı implementasyon

## Ortak Parametre Referansı

Tüm Connection, Device, Frame ve Variable parametrelerinin detaylı açıklamaları alt sayfalarda verilmektedir:

- [DNP3 Master](/docs/tr/protocols/dnp3/master/) — Master yapılandırması ve parametre detayları
- [DNP3 Outstation](/docs/tr/protocols/dnp3/outstation/) — Slave yapılandırması
