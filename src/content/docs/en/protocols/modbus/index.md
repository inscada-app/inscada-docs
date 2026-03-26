---
title: "Modbus"
description: "Modbus protocol family in inSCADA — TCP, RTU over TCP, UDP variants"
sidebar:
  order: 1
---

Modbus is one of the most widely used communication protocols in industrial automation. inSCADA supports multiple variants of the Modbus protocol in both **Client (Master)** and **Server (Slave)** roles.

## Supported Variants

| Variant | Client/Master | Server/Slave |
|---------|:------------:|:------------:|
| [Modbus TCP](/docs/en/protocols/modbus/tcp-client/) | ✓ | ✓ |
| [Modbus RTU over TCP](/docs/en/protocols/modbus/rtu-over-tcp/) | ✓ | ✓ |
| [Modbus UDP](/docs/en/protocols/modbus/udp/) | ✓ | ✓ |

## Common Concepts

Core concepts common to all Modbus variants:

- **Connection**: Protocol connection definition (IP, port, timeout, etc.)
- **Device**: Target device identified by Slave address (Unit ID)
- **Frame**: Register block definition (Coil, Discrete Input, Holding Register, Input Register)
- **Variable**: Single register or bit address, data type and scaling information

### Register Types

| Type | Function Code | Read | Write |
|------|--------------|:----:|:-----:|
| **Coil** | FC01 / FC05 / FC15 | ✓ | ✓ |
| **Discrete Input** | FC02 | ✓ | — |
| **Holding Register** | FC03 / FC06 / FC16 | ✓ | ✓ |
| **Input Register** | FC04 | ✓ | — |

Go to the relevant variant page for details.
