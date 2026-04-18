---
title: "MODBUS RTU over TCP"
description: "MODBUS RTU over TCP Master and Slave connection configuration"
sidebar:
  order: 2
  label: "MODBUS RTU over TCP"
---

MODBUS RTU is a protocol developed for the serial communication layer. Unlike MODBUS TCP, it does not contain an MBAP header. It is used for communication with serial port devices (RS232 or RS485) using the MODBUS RTU protocol. Since it operates at the TCP/IP application layer, it is used together with **transparent serial-ethernet converters** (terminal servers).

## How It Works

```
inSCADA ──(Ethernet)──► Terminal Server ──(RS485/RS232)──► Field Device
 (Master)              (Serial-Ethernet                    (Slave)
                        converter)
```

inSCADA connects to the terminal server as a Master using the MODBUS RTU over TCP protocol. The terminal server performs Ethernet-to-serial conversion and transmits protocol messages to the field device. This enables access to serial port devices over Ethernet.

## Configuration

MODBUS RTU over TCP configuration follows the same steps as [MODBUS TCP](/docs/en/protocols/modbus/tcp-client/). The only difference is selecting **MODBUS RTU Over TCP** as the protocol when creating the connection.

| Parameter | Difference |
|-----------|-----------|
| **Protocol** | Select `Modbus RTU Over TCP` |
| **Check CRC** | CRC checking recommended for RTU communication |
| All other parameters | Same as MODBUS TCP |

:::note
Make sure the terminal server is operating in transparent mode. Some terminal servers perform their own protocol conversion — in this case you may need to use MODBUS TCP instead.
:::

## MODBUS RTU over TCP Slave

inSCADA can also operate in MODBUS RTU over TCP Slave role. In this mode, external systems can connect to inSCADA via a terminal server and read data.

Select **Modbus RTU Over TCP Slave** as the protocol for configuration.
