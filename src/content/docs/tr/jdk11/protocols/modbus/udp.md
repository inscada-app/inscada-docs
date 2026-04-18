---
title: "MODBUS UDP"
description: "MODBUS UDP Client ve Slave bağlantı yapılandırması"
sidebar:
  order: 3
  label: "MODBUS UDP"
---

MODBUS UDP, MODBUS TCP'nin UDP transport katmanı üzerinden çalışan varyantıdır. TCP'ye göre daha düşük gecikme süresi sunar ancak bağlantı güvenilirliği garanti edilmez (connectionless).

## Ne Zaman Kullanılır?

- Düşük gecikme gereksinimi olan uygulamalar
- Basit, hafif haberleşme ihtiyacı
- Multicast veri dağıtımı senaryoları
- TCP bağlantı yükünün kabul edilemez olduğu durumlar

## Yapılandırma

MODBUS UDP yapılandırması, [MODBUS TCP](/docs/tr/protocols/modbus/tcp-client/) ile aynı adımları izler. Tek fark, bağlantı oluştururken protokol olarak **MODBUS UDP** seçilmesidir.

| Parametre | Fark |
|-----------|------|
| **Protocol** | `Modbus UDP` seçilir |
| **Port** | Varsayılan: 502 |
| Diğer tüm parametreler | MODBUS TCP ile aynı |

## MODBUS UDP Slave

inSCADA, MODBUS UDP Slave rolünde de çalışabilir. Bu modda dış sistemlerin UDP üzerinden inSCADA'ya bağlanıp veri okumasına izin verilir.

Yapılandırma için protokol olarak **Modbus UDP Slave** seçin.

:::caution
UDP, paket kaybı veya sıra değişikliği durumlarında yeniden iletim yapmaz. Kritik kontrol uygulamalarında MODBUS TCP tercih edilmelidir.
:::
