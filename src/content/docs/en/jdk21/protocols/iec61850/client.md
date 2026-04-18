---
title: "IEC 61850 Client"
description: "inSCADA'da IEC 61850 MMS Client bağlantı yapılandırması"
sidebar:
  order: 1
  label: "IEC 61850 Client"
---

Client modunda inSCADA, IED'lere (koruma rölesi, ölçüm cihazı, kesici kontrol ünitesi vb.) MMS üzerinden bağlanarak veri okur ve kontrol komutları gönderir.

## Adım 1: Connection Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | IEC 61850 | Protokol seçimi |
| **IP Address** | 192.168.1.50 | Hedef IED IP adresi |
| **Port** | 102 | MMS portu (varsayılan: 102) |
| **Local IP** | (boş) | Yerel ağ adaptörü IP adresi (çoklu NIC için) |
| **Local Port** | (boş) | Yerel port (opsiyonel) |
| **Response Timeout** | 5000 ms | Yanıt bekleme süresi |
| **Message Fragment Timeout** | 5000 ms | Mesaj parça timeout'u |
| **Authentication Parameter** | (boş) | Kimlik doğrulama parametresi (opsiyonel) |
| **SCL File Content** | (dosya içeriği) | IED'in SCL dosyası (.icd/.scd/.cid) |
| **Server Model Index** | 0 | SCL dosyasındaki server model indeksi |

### SCL Dosyası Import

IEC 61850 bağlantısının en kritik adımı SCL dosyasının yüklenmesidir. SCL dosyası, IED'in tüm veri modelini (Logical Device, Logical Node, Data Object, Data Attribute) tanımlar.

1. IED üreticisinden SCL dosyasını (.icd, .scd veya .cid) temin edin
2. Connection oluştururken SCL dosyasını platforma yükleyin
3. inSCADA, SCL dosyasını parse ederek cihazın ServerModel'ini çıkarır

:::tip
SCL dosyası olmadan IEC 61850 bağlantısı kurulamaz. IED'i devreye almadan önce üreticiden güncel SCL dosyasını talep edin.
:::

## Adım 2: Device Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Object Reference** | `IED1LD1` | Logical Device referansı (SCL'den) |
| **Scan Time** | 1000 ms | Tarama periyodu |
| **Scan Type** | PERIODIC | `PERIODIC` veya `FIXED_DELAY` |

## Adım 3: Frame Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Object Reference** | `IED1LD1/MMXU1` | Logical Node referansı |

Frame, bir Logical Node'u temsil eder. SCL dosyasındaki her Logical Node (XCBR, XSWI, MMXU vb.) için bir Frame oluşturulur.

### Tipik Frame Örnekleri

| Frame Adı | Object Reference | Açıklama |
|-----------|-----------------|----------|
| Kesici | `IED1LD1/XCBR1` | Circuit Breaker durumu ve kontrolü |
| Ayırıcı | `IED1LD1/XSWI1` | Disconnector durumu |
| Ölçüm | `IED1LD1/MMXU1` | Gerilim, akım, güç ölçümleri |
| Koruma | `IED1LD1/PTOC1` | Aşırı akım koruma bilgileri |
| Temel | `IED1LD1/LLN0` | Lojik cihaz temel bilgileri |

## Adım 4: Variable Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Object Reference** | `IED1LD1/MMXU1.PhV.phsA.cVal.mag.f` | Tam veri özniteliği referansı |
| **FC** | MX | Functional Constraint |
| **Type** | Float32 | Veri tipi |

### Okuma ve Yazma

Variable'ın FC değeri işlemin türünü belirler:

| FC | İşlem | Örnek |
|----|-------|-------|
| **ST** | Okuma — durum bilgisi | `XCBR1.Pos.stVal` (kesici pozisyonu) |
| **MX** | Okuma — ölçüm değeri | `MMXU1.PhV.phsA.cVal.mag.f` (faz-A gerilim) |
| **CO** | Yazma — kontrol komutu | `XCBR1.Pos` (kesici açma/kapama) |
| **SP** | Yazma — setpoint | Ayar değeri gönderme |

### Kontrol Modları

IEC 61850, kontrol komutları için iki mod destekler:

- **Direct Operate:** Doğrudan komut gönderme — hızlı ama onaysız
- **SBO (Select Before Operate):** Önce seçim, sonra çalıştırma — güvenli kontrol

## Adım 5: Bağlantıyı Başlatma

**Runtime Control Panel**'den bağlantıyı başlatın. inSCADA:
1. TCP bağlantısı kurar (port 102)
2. MMS association oluşturur
3. SCL'den ServerModel'i yükler
4. Periyodik veri okumaya başlar

Bağlantı durumu "Connected" olarak görünecektir.
