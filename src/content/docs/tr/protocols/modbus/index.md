---
title: "Modbus"
description: "inSCADA'da Modbus protokol ailesi — TCP, RTU over TCP, UDP varyantları"
sidebar:
  order: 1
---

Modbus, endüstriyel otomasyonda en yaygın kullanılan haberleşme protokollerinden biridir. inSCADA, Modbus protokolünün birden fazla varyantını hem **Client (Master)** hem de **Server (Slave)** rolünde destekler.

## Desteklenen Varyantlar

| Varyant | Client/Master | Server/Slave |
|---------|:------------:|:------------:|
| [Modbus TCP](/docs/tr/protocols/modbus/tcp-client/) | ✓ | ✓ |
| [Modbus RTU over TCP](/docs/tr/protocols/modbus/rtu-over-tcp/) | ✓ | ✓ |
| [Modbus UDP](/docs/tr/protocols/modbus/udp/) | ✓ | ✓ |

## Ortak Kavramlar

Tüm Modbus varyantlarında ortak olan temel kavramlar:

- **Connection**: Protokol bağlantı tanımı (IP, port, timeout vb.)
- **Device**: Slave adresi (Unit ID) ile tanımlanan hedef cihaz
- **Frame**: Register bloğu tanımı (Coil, Discrete Input, Holding Register, Input Register)
- **Variable**: Tek bir register veya bit adresi, veri tipi ve ölçekleme bilgisi

### Register Tipleri

| Tip | Fonksiyon Kodu | Okuma | Yazma |
|-----|---------------|:-----:|:-----:|
| **Coil** | FC01 / FC05 / FC15 | ✓ | ✓ |
| **Discrete Input** | FC02 | ✓ | — |
| **Holding Register** | FC03 / FC06 / FC16 | ✓ | ✓ |
| **Input Register** | FC04 | ✓ | — |

Detaylar için ilgili varyantın sayfasına gidin.
