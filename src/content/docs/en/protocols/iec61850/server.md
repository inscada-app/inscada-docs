---
title: "IEC 61850 Server"
description: "inSCADA'da IEC 61850 MMS Server yapılandırması"
sidebar:
  order: 2
  label: "IEC 61850 Server"
---

Server modunda inSCADA, dış IEC 61850 Client'ların (üst SCADA, kontrol merkezi, başka bir IED) bağlanıp MMS üzerinden veri okumasına izin verir.

## Kullanım Senaryoları

- Üst SCADA sistemlerine IEC 61850 MMS üzerinden veri sunmak
- Farklı inSCADA instance'ları arasında IEC 61850 haberleşmesi
- Trafo merkezi otomasyonunda gateway rolü — farklı protokollerden toplanan verileri IEC 61850 formatında sunmak

## Yapılandırma

### Connection

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | IEC 61850 Server | Protokol seçimi |
| **Port** | 102 | Dinleme portu (gelen bağlantılar için) |
| **SCL File Content** | (dosya içeriği) | Sunulacak veri modelini tanımlayan SCL dosyası |

:::caution
Server portu, firewall'da **gelen bağlantılara açık** olmalıdır.
:::

### SCL Dosyası

Server modunda SCL dosyası, inSCADA'nın dış dünyaya sunacağı veri modelini tanımlar. Bu SCL dosyası:
- Hangi Logical Device'ların sunulacağını
- Her LD altındaki Logical Node'ları
- Her LN altındaki Data Object ve Data Attribute'ları
belirler.

### Device, Frame ve Variable

Tanım yapısı Client moduyla aynıdır. Tanımlanan değişkenlerin değerleri, bağlanan Client'lara MMS üzerinden sunulur.

## Kullanım Senaryosu

```
Saha Cihazları (Röle, RTU, Ölçüm)
    │
    │ (MODBUS, IEC 104, DNP3 vb.)
    ▼
┌──────────────────────┐
│   inSCADA              │
│   (Veri Toplama)       │
│                        │
│   IEC 61850 Server     │
│   (MMS üzerinden)      │
└────────────┬───────────┘
             │
             │ MMS (port 102)
             ▼
┌──────────────────────┐
│   Üst SCADA /          │
│   Kontrol Merkezi      │
│   (IEC 61850 Client)   │
└──────────────────────┘
```

Bu senaryoda inSCADA, saha cihazlarından farklı protokollerle veri toplar ve bu verileri IEC 61850 MMS Server olarak üst sistemlere sunar. Özellikle trafo merkezi otomasyonunda farklı üreticilerin IED'lerinden toplanan verilerin standart bir formatta sunulması için kullanılır.
