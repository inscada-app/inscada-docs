---
title: "OPC UA Server"
description: "inSCADA'da OPC UA Server yapılandırması"
sidebar:
  order: 2
  label: "OPC UA Server"
---

Server modunda inSCADA, dış OPC UA Client'ların bağlanıp veri okumasına ve yazmasına izin verir. inSCADA kendi namespace'ini oluşturarak toplanan verileri OPC UA node'ları olarak sunar.

## Kullanım Senaryoları

- Üst SCADA veya MES/ERP sistemlerine OPC UA üzerinden veri sunmak
- Farklı inSCADA instance'ları arasında OPC UA haberleşmesi
- Üçüncü parti analitik ve raporlama araçlarına veri sağlamak
- Bulut platformlarına (Azure IoT Hub, AWS IoT vb.) OPC UA gateway olarak çalışmak

## Yapılandırma

### Connection

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | OPC UA Server | Protokol seçimi |
| **Port** | 4840 | Dinleme portu (gelen bağlantılar için) |
| **Security Mode** | SignAndEncrypt | Güvenlik modu |
| **Security Policy** | Basic256Sha256 | Güvenlik politikası |
| **Encoding Type** | Binary | Encoding tipi |

Güvenlik ve kimlik doğrulama ayarları Client ile aynı yapıdadır.

:::caution
OPC UA Server portu, firewall'da **gelen bağlantılara açık** olmalıdır.
:::

### Namespace

inSCADA, OPC UA Server için otomatik bir namespace oluşturur: `urn:inscada:milo:connection-namespace`. Tanımlanan değişkenler bu namespace altında node olarak sunulur.

### Device, Frame ve Variable

Tanım yapısı Client moduyla aynıdır. Tanımlanan değişkenlerin değerleri, bağlanan OPC UA Client'lara node olarak sunulur.

## Kullanım Senaryosu

```
Saha Cihazları
    │
    │ (MODBUS, IEC 104, DNP3, S7 vb.)
    ▼
┌──────────────────────┐
│   inSCADA              │
│   (Veri Toplama)       │
│                        │
│   OPC UA Server        │
│   (port 4840)          │
└────────────┬───────────┘
             │
             │ OPC UA
             ▼
┌──────────────────────┐
│   Üst SCADA / MES /   │
│   ERP / Bulut          │
│   (OPC UA Client)      │
└──────────────────────┘
```

Bu senaryoda inSCADA, saha cihazlarından farklı protokollerle veri toplar ve bu verileri OPC UA Server olarak üst sistemlere sunar. OPC UA'nın platform bağımsız yapısı sayesinde farklı işletim sistemleri ve ortamlar arasında sorunsuz entegrasyon sağlanır.
