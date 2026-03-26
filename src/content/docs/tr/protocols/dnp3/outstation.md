---
title: "DNP3 Outstation"
description: "inSCADA'da DNP3 Outstation (Slave) yapılandırması"
sidebar:
  order: 2
  label: "DNP3 Outstation"
---

Outstation modunda inSCADA, dış Master'ların bağlanıp veri okumasına izin verir. Bu mod genellikle:

- Üst SCADA sistemlerine DNP3 üzerinden veri sunmak
- Kontrol merkezine veri aktarmak
- Farklı inSCADA instance'ları arasında veri paylaşımı

için kullanılır.

## Yapılandırma

### Connection

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | DNP3 Slave | Protokol seçimi |
| **Port** | 20000 | Dinleme portu (gelen bağlantılar için) |

:::caution
Outstation portu, firewall'da **gelen bağlantılara açık** olmalıdır.
:::

### Device

Outstation modunda adres rolleri **tersine döner:**

| Parametre | Açıklama |
|-----------|----------|
| **Local Address** | Outstation'ın kendi DNP3 adresi |
| **Remote Address** | Bağlanacak Master'ın DNP3 adresi |

### Frame ve Variable

Frame ve Variable tanımları Master modundaki ile aynı yapıdadır. Tanımlanan değişkenlerin değerleri, bağlanan Master'a DNP3 object'leri olarak sunulur.

## Kullanım Senaryosu

```
Saha Cihazları
    │
    │ (MODBUS, IEC 104, vb.)
    ▼
┌──────────────────┐
│   inSCADA         │
│   (Veri Toplama)  │
└────────┬─────────┘
         │
         │ DNP3 Outstation
         ▼
┌──────────────────┐
│   Üst SCADA /     │
│   Kontrol Merkezi  │
│   (DNP3 Master)    │
└──────────────────┘
```

Bu senaryoda inSCADA, saha cihazlarından farklı protokollerle veri toplar ve bu verileri DNP3 Outstation olarak üst sistemlere sunar. Protokol dönüştürücü (gateway) görevi üstlenir.
