---
title: "MODBUS UDP"
description: "MODBUS UDP Client and Slave connection configuration"
sidebar:
  order: 3
  label: "MODBUS UDP"
---

MODBUS UDP is a variant of MODBUS TCP that operates over the UDP transport layer. It offers lower latency than TCP but does not guarantee connection reliability (connectionless).

## When to Use

- Applications requiring low latency
- Simple, lightweight communication needs
- Multicast data distribution scenarios
- Situations where TCP connection overhead is unacceptable

## Configuration

MODBUS UDP configuration follows the same steps as [MODBUS TCP](/docs/en/protocols/modbus/tcp-client/). The only difference is selecting **MODBUS UDP** as the protocol when creating the connection.

| Parameter | Difference |
|-----------|-----------|
| **Protocol** | Select `Modbus UDP` |
| **Port** | Default: 502 |
| All other parameters | Same as MODBUS TCP |

## MODBUS UDP Slave

inSCADA can also operate in MODBUS UDP Slave role. In this mode, external systems can connect to inSCADA via UDP and read data.

Select **Modbus UDP Slave** as the protocol for configuration.

:::caution
UDP does not retransmit in case of packet loss or reordering. MODBUS TCP should be preferred for critical control applications.
:::
