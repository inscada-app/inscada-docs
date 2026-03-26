---
title: "MODBUS TCP"
description: "MODBUS TCP Client ve Server bağlantı yapılandırması"
sidebar:
  order: 1
  label: "MODBUS TCP"
---

MODBUS TCP, Ethernet üzerinden MODBUS haberleşmesi sağlayan en yaygın kullanılan varyantdır. TCP/IP katmanı üzerinde çalışır ve varsayılan olarak port **502** kullanır. inSCADA, MODBUS TCP protokolünü hem **Client (Master)** hem de **Server (Slave)** rolünde destekler.

## MODBUS TCP Client (Master)

Client modunda inSCADA, saha cihazlarına (PLC, RTU, enerji analizörü vb.) bağlanarak veri okur ve yazar.

### Adım 1: Connection Oluşturma

Öncelikle hedef cihaza bir bağlantı tanımlayın.

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | Modbus TCP | Protokol seçimi |
| **IP Address** | 192.168.1.100 | Hedef cihaz IP adresi |
| **Port** | 502 | Hedef port (varsayılan: 502) |
| **Timeout** | 3000 ms | Yanıt bekleme süresi |
| **Connect Timeout** | 5000 ms | Bağlantı kurma zaman aşımı |
| **Retries** | 3 | Başarısız istek tekrar sayısı |
| **Reconnect on Error** | true | Hata durumunda otomatik yeniden bağlanma |

### Adım 2: Device Oluşturma

Bağlantı altına bir veya birden fazla cihaz ekleyin. Her cihaz farklı bir Slave adresi (Unit ID) temsil eder.

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Station Address** | 1 | MODBUS Slave adresi (1-247) |
| **Scan Time** | 1000 ms | Tarama periyodu |
| **Scan Type** | PERIODIC | `PERIODIC` veya `FIXED_DELAY` |

### Adım 3: Frame (Veri Bloğu) Oluşturma

Cihazdan okunacak register bloklarını tanımlayın. MODBUS veri blokları maksimum **255 byte**, **127 Word** veya **63 Double Word** boyutunda olabilir.

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Type** | Holding Register | Register tipi |
| **Start Address** | 1000 | Başlangıç adresi |
| **Quantity** | 20 | Okunacak register sayısı |
| **Is Readable** | true | Okuma izni |
| **Is Writable** | true | Yazma izni |

:::tip
Büyük register aralıklarını birden fazla Frame'e bölün. Bu, haberleşme performansını artırır ve hata durumunda sadece ilgili bloğu etkiler.
:::

### Adım 4: Variable Oluşturma

Frame içindeki her bir veri noktasını değişken olarak tanımlayın.

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Start Address** | 0 | Frame içindeki register ofseti |
| **Type** | Float | Veri tipi |
| **Byte Swap** | false | Byte sırası değiştirme |
| **Word Swap** | false | Word sırası değiştirme |

### Adım 5: Bağlantıyı Başlatma

Yapılandırma tamamlandıktan sonra **Runtime Control Panel**'den bağlantıyı başlatın. Bağlantı durumu "Connected" olarak görünecektir.

## MODBUS TCP Server (Slave)

Server modunda inSCADA, dış sistemlerin (SCADA, DCS, başka bir inSCADA instance'ı vb.) bağlanıp veri okumasına izin verir. Bu mod, inSCADA'nın topladığı verileri üst sistemlere sunmak için kullanılır.

### Yapılandırma

Server bağlantısı oluştururken:

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | Modbus TCP Slave | Protokol seçimi |
| **Port** | 502 | Dinleme portu (gelen bağlantılar için) |

:::caution
Server portu, firewall'da **gelen bağlantılara açık** olmalıdır. Aynı portu kullanan başka bir servis olmadığından emin olun.
:::

### Veri Sunma

Server modunda Frame ve Variable tanımları, Client modundaki ile aynı yapıdadır. Tanımlanan değişkenlerin değerleri, bağlanan client'lara MODBUS register'ları olarak sunulur.

## Sonraki Adımlar

- [MODBUS RTU over TCP](/docs/tr/protocols/modbus/rtu-over-tcp/) — Seri port cihazlarla terminal server üzerinden haberleşme
- [MODBUS UDP](/docs/tr/protocols/modbus/udp/) — UDP tabanlı MODBUS haberleşme
- [MODBUS genel parametreler](/docs/tr/protocols/modbus/) — Veri tipleri, register tipleri ve ortak kavramlar
