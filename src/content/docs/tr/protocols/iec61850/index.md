---
title: "IEC 61850"
description: "inSCADA'da IEC 61850 MMS protokolü — genel bakış, veri modeli ve ortak parametreler"
sidebar:
  order: 0
  label: "IEC 61850"
---

IEC 61850, trafo merkezleri ve enerji sistemlerinde haberleşme için geliştirilmiş uluslararası bir standarttır. Geleneksel protokollerden farklı olarak, yalnızca bir haberleşme protokolü değil aynı zamanda bir **veri modelleme standardıdır** — cihazların fonksiyonlarını, verilerini ve davranışlarını standart bir yapıda tanımlar.

inSCADA, IEC 61850 standardının **MMS (Manufacturing Message Specification)** haberleşme katmanını hem **Client** hem de **Server** rolünde destekler.

:::note
inSCADA JDK 11 Edition'da **GOOSE** ve **Sampled Values** desteklenmemektedir. Yalnızca MMS üzerinden haberleşme sağlanır.
:::

## Standart Hakkında

### Kullanım Alanları

- Trafo merkezleri (primer alan)
- Enerji üretim santralleri
- Dağıtım otomasyonu
- Rüzgar ve güneş enerjisi santralleri
- Endüstriyel enerji yönetimi

### Temel Avantajlar

- **Üretici bağımsız:** Farklı üreticilerin IED'leri standart veri modeli ile birlikte çalışır
- **Kendi kendini tanımlayan:** SCL dosyası ile cihaz yapılandırması otomatik keşfedilebilir
- **Nesne yönelimli veri modeli:** Fiziksel cihaz fonksiyonları lojik düğümlerle temsil edilir
- **Yüksek performans:** MMS üzerinden hızlı veri erişimi

## IEC 61850 Veri Modeli

IEC 61850, fiziksel cihazları hiyerarşik bir lojik yapıyla modeller:

```
Physical Device (Fiziksel Cihaz — IED)
└── Logical Device (Lojik Cihaz — LD)
    └── Logical Node (Lojik Düğüm — LN)
        └── Data Object (Veri Nesnesi — DO)
            └── Data Attribute (Veri Özniteliği — DA)
```

**Örnek Object Reference:** `IED1/LLN0.Mod.stVal`
- `IED1` — Fiziksel cihaz
- `LLN0` — Logical Node (LN0: temel düğüm)
- `Mod` — Data Object (Mode — çalışma modu)
- `stVal` — Data Attribute (status value — durum değeri)

### Logical Node Örnekleri

| Logical Node | Açıklama |
|-------------|----------|
| **LLN0** | Lojik cihaz temel düğümü |
| **LPHD** | Fiziksel cihaz bilgisi |
| **XCBR** | Kesici (Circuit Breaker) |
| **XSWI** | Ayırıcı (Disconnector) |
| **MMXU** | Ölçüm birimi (gerilim, akım, güç) |
| **CSWI** | Anahtarlama kontrolü |
| **PTOC** | Aşırı akım koruma |
| **PDIS** | Mesafe koruma |

### Functional Constraint (FC)

Her Data Attribute bir **Functional Constraint** ile sınıflandırılır. FC, verinin amacını belirtir:

| FC | Açıklama | Kullanım |
|----|----------|----------|
| **ST** | Status Information | Durum bilgileri okuma |
| **MX** | Measurands (Analog Values) | Ölçüm değerleri okuma |
| **CO** | Control | Kumanda yazma |
| **SP** | Setpoint | Ayar değeri yazma |
| **SE** | Setting Group Editable | Ayar grubu düzenleme |
| **CF** | Configuration | Yapılandırma bilgileri |
| **DC** | Description | Açıklama bilgileri |
| **SG** | Setting Group | Ayar grubu |
| **SV** | Substitution | Yerine koyma |
| **BL** | Blocking | Bloklama |
| **OR** | Operate Received | İşlem alındı bilgisi |
| **RP** | Unbuffered Reporting | Tamponlanmamış raporlama |
| **BR** | Buffered Reporting | Tamponlanmış raporlama |
| **EX** | Extended Definition | Genişletilmiş tanım |
| **SR** | Service Response | Servis yanıtı / takip |

:::tip
Tipik SCADA uygulamalarında en çok kullanılan FC'ler: **ST** (durum okuma), **MX** (ölçüm okuma) ve **CO** (kontrol yazma).
:::

### SCL (Substation Configuration Language)

IEC 61850, cihaz yapılandırmasını **SCL** dosyalarıyla tanımlar. SCL, XML tabanlı bir format olup cihazın tüm lojik yapısını (LD, LN, DO, DA) içerir.

SCL dosya uzantıları:
- **.icd** — IED Capability Description (cihaz yeteneği)
- **.scd** — Substation Configuration Description (tesis yapılandırması)
- **.cid** — Configured IED Description (yapılandırılmış IED)

inSCADA, bağlantı oluşturulurken SCL dosyasını import ederek cihazın veri modelini otomatik olarak keşfeder.

## inSCADA Veri Modeli

inSCADA'da IEC 61850 bağlantısı şu hiyerarşide yapılandırılır:

```
Connection (Bağlantı — IP, port, SCL dosyası)
└── Device (Cihaz — Object Reference ile Logical Device)
    └── Frame (Veri Bloğu — Object Reference ile Logical Node)
        └── Variable (Değişken — Object Reference + FC)
```

### Desteklenen Veri Tipleri

| Veri Tipi | Açıklama |
|-----------|----------|
| **Boolean** | Tek bit değer |
| **Int8 / UInt8** | 8-bit tam sayı |
| **Int16 / UInt16** | 16-bit tam sayı |
| **Int32 / UInt32** | 32-bit tam sayı |
| **Int64** | 64-bit tam sayı |
| **Int128** | 128-bit tam sayı |
| **Float32** | 32-bit kayan nokta |
| **Float64** | 64-bit kayan nokta |
| **Visible String** | ASCII karakter dizisi |
| **Unicode String** | Unicode karakter dizisi |
| **Octet String** | Byte dizisi |
| **Timestamp** | Zaman damgası |
| **Quality** | Kalite bilgisi |
| **Double Bit Pos** | Çift bit pozisyon (kesici/ayırıcı durumu) |
| **Check** | Kontrol doğrulama |
| **Tap Command** | Kademe komutu |

## Alt Sayfalar

- [IEC 61850 Client](/docs/tr/protocols/iec61850/client/) — MMS Client yapılandırması
- [IEC 61850 Server](/docs/tr/protocols/iec61850/server/) — MMS Server yapılandırması
