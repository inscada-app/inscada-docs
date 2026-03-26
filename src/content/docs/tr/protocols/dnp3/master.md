---
title: "DNP3 Master"
description: "inSCADA'da DNP3 Master bağlantı yapılandırması"
sidebar:
  order: 1
  label: "DNP3 Master"
---

Master modunda inSCADA, saha cihazlarına (RTU, IED, relay vb.) bağlanarak veri okur ve kontrol komutları gönderir.

## Veri Modeli

```
Connection (Bağlantı)
└── Device (Cihaz — Local/Remote adres çifti)
    └── Frame (Veri Bloğu — Object group tanımı)
        └── Variable (Değişken — Point index)
```

## Adım 1: Connection Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | DNP3 | Protokol seçimi |
| **IP Address** | 192.168.1.50 | Hedef Outstation IP adresi |
| **Port** | 20000 | Hedef port (varsayılan: 20000) |
| **Adapter** | (boş) | Ağ adaptörü (çoklu NIC için) |
| **Pool Size** | 1 | Bağlantı havuzu boyutu |
| **Min Retry Delay** | 1000 ms | Minimum yeniden bağlanma bekleme süresi |
| **Max Retry Delay** | 60000 ms | Maksimum yeniden bağlanma bekleme süresi |

## Adım 2: Device Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Local Address** | 1 | Master'ın DNP3 adresi |
| **Remote Address** | 10 | Outstation'ın DNP3 adresi |
| **Response Timeout** | 5000 ms | Yanıt bekleme süresi |
| **Integrity Scan Time** | 60000 ms | Bütünlük tarama periyodu (tüm static) |
| **Event Scan Time** | 5000 ms | Olay tarama periyodu (Class 1/2/3) |
| **Scan Type** | PERIODIC | `PERIODIC` veya `FIXED_DELAY` |
| **Unsolicited Events** | true | Unsolicited response kabul |
| **Disable Unsolicited on Startup** | false | Başlangıçta unsolicited devre dışı |
| **Startup Integrity** | true | Başlangıçta bütünlük taraması |
| **Integrity on Event Overflow IIN** | true | Buffer taşmasında bütünlük taraması |

:::tip
**Integrity Scan** tüm static verileri sorgular ve Outstation veritabanının tam kopyasını alır. **Event Scan** yalnızca son taramadan bu yana değişen verileri getirir.
:::

## Adım 3: Frame Oluşturma

Her veri grubu için bir Frame tanımlayın:

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Type** | Analog Input | Veri tipi |
| **Start Address** | 0 | Başlangıç point index |
| **Quantity** | 10 | Point sayısı |
| **Event Buffer Size** | 100 | Olay tamponu boyutu |
| **Point Class** | Class 1 | Olay sınıfı |
| **Static Variation** | Group30Var5 | 32-bit float (flag'li) |
| **Event Variation** | Group32Var7 | 32-bit float (zaman damgalı) |
| **Deadband** | 0.5 | Analog olay eşiği |

### Tipik Frame Örnekleri

| Frame Adı | Type | Start | Quantity | Açıklama |
|-----------|------|-------|----------|----------|
| Analog Inputs | Analog Input | 0 | 20 | Sıcaklık, basınç, akım ölçümleri |
| Binary Inputs | Binary Input | 0 | 32 | Dijital durum bilgileri |
| Counters | Counter | 0 | 8 | Enerji sayaçları |
| Binary Outputs | Binary Output | 0 | 16 | Kontrol çıkışları |

## Adım 4: Variable Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Start Address** | 0 | Point index |
| **Point Class** | Class 1 | Olay sınıfı (frame'den miras alabilir) |
| **Static Variation** | Group30Var5 | Static veri formatı |
| **Event Variation** | Group32Var7 | Event veri formatı |
| **Deadband** | 0.5 | Analog olay eşiği |

## Adım 5: Bağlantıyı Başlatma

**Runtime Control Panel**'den bağlantıyı başlatın. Bağlantı durumu "Connected" olarak görünecektir.

## Kontrol Komutları

Master, Outstation'a şu komutları gönderebilir:

- **Select-Before-Operate (SBO):** Önce seçim, sonra çalıştırma — güvenli kontrol
- **Direct Operate:** Doğrudan çalıştırma
- **Freeze Accumulator:** Sayaç dondurma
- **Zaman Senkronizasyonu:** Outstation saatini ayarlama
