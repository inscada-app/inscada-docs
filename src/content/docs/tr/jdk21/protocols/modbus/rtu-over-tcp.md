---
title: "MODBUS RTU over TCP"
description: "MODBUS RTU over TCP Master ve Slave bağlantı yapılandırması"
sidebar:
  order: 2
  label: "MODBUS RTU over TCP"
---

MODBUS RTU, seri haberleşme katmanı için geliştirilmiş bir protokoldür. MODBUS TCP'den farklı olarak MBAP header içermez. RS232 veya RS485 seri port cihazları ile MODBUS RTU protokolünü kullanarak haberleşme sağlar.

TCP/IP uygulama katmanında çalıştığı için, **transparent seri-ethernet dönüştürücüler** (terminal server) ile birlikte kullanılır.

## Çalışma Prensibi

```
inSCADA ──(Ethernet)──► Terminal Server ──(RS485/RS232)──► Saha Cihazı
 (Master)              (Seri-Ethernet                      (Slave)
                        dönüştürücü)
```

inSCADA, terminal server'a MODBUS RTU over TCP protokolü ile Master olarak bağlanır. Terminal server, Ethernet-seri dönüşümü yaparak protokol mesajlarını saha cihazına iletir. Bu sayede seri port cihazlarına Ethernet üzerinden erişim sağlanır.

## Yapılandırma

MODBUS RTU over TCP yapılandırması, [MODBUS TCP](/docs/tr/protocols/modbus/tcp-client/) ile aynı adımları izler. Tek fark, bağlantı oluştururken protokol olarak **MODBUS RTU Over TCP** seçilmesidir.

| Parametre | Fark |
|-----------|------|
| **Protocol** | `Modbus RTU Over TCP` seçilir |
| **Check CRC** | RTU haberleşmede CRC kontrolü önerilir |
| Diğer tüm parametreler | MODBUS TCP ile aynı |

:::note
Terminal server'ın transparent (şeffaf) modda çalıştığından emin olun. Bazı terminal server'lar kendi protokol dönüşümü yapabilir — bu durumda MODBUS TCP kullanmanız gerekebilir.
:::

## MODBUS RTU over TCP Slave

inSCADA, MODBUS RTU over TCP Slave rolünde de çalışabilir. Bu modda dış sistemlerin terminal server üzerinden inSCADA'ya bağlanıp veri okumasına izin verilir.

Yapılandırma için protokol olarak **Modbus RTU Over TCP Slave** seçin.
