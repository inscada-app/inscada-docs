---
title: "MODBUS TCP"
description: "MODBUS TCP Client and Server connection configuration"
sidebar:
  order: 1
  label: "MODBUS TCP"
---

MODBUS TCP is the most commonly used variant for MODBUS communication over Ethernet. It operates on the TCP/IP layer and uses port **502** by default. inSCADA supports the MODBUS TCP protocol in both **Client (Master)** and **Server (Slave)** roles.

## MODBUS TCP Client (Master)

In Client mode, inSCADA connects to field devices (PLCs, RTUs, energy analysers, etc.) to read and write data.

### Step 1: Create a Connection

First, define a connection to the target device.

| Parameter | Example | Description |
|-----------|---------|-------------|
| **Protocol** | Modbus TCP | Protocol selection |
| **IP Address** | 192.168.1.100 | Target device IP address |
| **Port** | 502 | Target port (default: 502) |
| **Timeout** | 3000 ms | Response wait time |
| **Connect Timeout** | 5000 ms | Connection establishment timeout |
| **Retries** | 3 | Failed request retry count |
| **Reconnect on Error** | true | Automatic reconnection on error |

### Step 2: Create a Device

Add one or more devices under the connection. Each device represents a different Slave address (Unit ID).

| Parameter | Example | Description |
|-----------|---------|-------------|
| **Station Address** | 1 | MODBUS Slave address (1-247) |
| **Scan Time** | 1000 ms | Scan period |
| **Scan Type** | PERIODIC | `PERIODIC` or `FIXED_DELAY` |

### Step 3: Create a Frame (Data Block)

Define the register blocks to be read from the device. MODBUS data blocks can be a maximum of **255 bytes**, **127 Words** or **63 Double Words**.

| Parameter | Example | Description |
|-----------|---------|-------------|
| **Type** | Holding Register | Register type |
| **Start Address** | 1000 | Starting address |
| **Quantity** | 20 | Number of registers to read |
| **Is Readable** | true | Read permission |
| **Is Writable** | true | Write permission |

:::tip
Split large register ranges into multiple Frames. This improves communication performance and limits the impact of errors to the affected block only.
:::

### Step 4: Create Variables

Define each data point within the Frame as a variable.

| Parameter | Example | Description |
|-----------|---------|-------------|
| **Start Address** | 0 | Register offset within frame |
| **Type** | Float | Data type |
| **Byte Swap** | false | Byte order swap |
| **Word Swap** | false | Word order swap |

### Step 5: Start the Connection

Once configuration is complete, start the connection from the **Runtime Control Panel**. The connection status will show "Connected".

## MODBUS TCP Server (Slave)

In Server mode, inSCADA allows external systems (SCADA, DCS, another inSCADA instance, etc.) to connect and read data. This mode is used to present data collected by inSCADA to upstream systems.

### Configuration

When creating a Server connection:

| Parameter | Example | Description |
|-----------|---------|-------------|
| **Protocol** | Modbus TCP Slave | Protocol selection |
| **Port** | 502 | Listening port (for incoming connections) |

:::caution
The Server port must be **open for inbound connections** in the firewall. Make sure no other service is using the same port.
:::

### Data Presentation

In Server mode, Frame and Variable definitions follow the same structure as Client mode. The values of defined variables are presented to connecting clients as MODBUS registers.

## Next Steps

- [MODBUS RTU over TCP](/docs/en/protocols/modbus/rtu-over-tcp/) — Communication with serial port devices via terminal server
- [MODBUS UDP](/docs/en/protocols/modbus/udp/) — UDP-based MODBUS communication
- [MODBUS common parameters](/docs/en/protocols/modbus/) — Data types, register types and common concepts
