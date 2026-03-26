---
title: "DNP3"
description: "DNP3 Master and Slave (Outstation) connection configuration in inSCADA"
sidebar:
  order: 2
---

DNP3 (Distributed Network Protocol) is a protocol developed to provide standards-based interoperability between control centre computers, RTUs and IEDs (Intelligent Electronic Devices) in the electrical utility, water/wastewater, transportation and oil/gas industries.

inSCADA supports the DNP3 protocol in both **Master** and **Slave (Outstation)** roles. It operates over TCP/IP.

## Core Concepts

### Master and Outstation

There are two fundamental roles in DNP3 communication:

- **Master:** The side that collects data by sending queries to Outstations. In Master role, inSCADA reads data from field devices and sends control commands.
- **Outstation (Slave):** The side that holds field data in its database and responds to Master queries. In Outstation role, inSCADA presents data to external systems.

### Data Types (Object Groups)

DNP3 classifies data types using **Group** and **Variation** numbers:

| Data Type | Description | Read | Write |
|-----------|-------------|:----:|:-----:|
| **Binary Input** | Digital input (open/closed) | ✓ | — |
| **Double Input** | Double-bit digital input | ✓ | — |
| **Binary Output** | Digital output (control) | ✓ | ✓ |
| **Counter** | Counter value (kWh, etc.) | ✓ | — |
| **Frozen Counter** | Frozen counter | ✓ | — |
| **Analog Input** | Analog measurement value | ✓ | — |
| **Analog Output** | Analog output (setpoint) | ✓ | ✓ |

### Static and Event Data

- **Static Data:** Current values — the present state of a binary input or the current analog measurement at the time of transmission
- **Event Data:** State changes, threshold crossings and similar occurrences. Can be reported with or without timestamps.

DNP3 classifies events into three classes:
- **Class 1:** Highest priority
- **Class 2:** Medium priority
- **Class 3:** Lowest priority

### Unsolicited Responses

Outstations can send change notifications without a Master query. This feature provides instant notification in systems with many outstations without waiting for polling cycles.

## Data Model

```
Connection
└── Device (Local/Remote address pair)
    └── Frame (Object group definition)
        └── Variable (Point index)
```

### Connection Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| **IP / Host** | String | Target device IP address |
| **Port** | Integer | Target port (default: 20000) |
| **Adapter** | String | Network adapter selection |
| **Pool Size** | Integer | Connection pool size |
| **Min Retry Delay** | Integer (ms) | Minimum reconnection wait time |
| **Max Retry Delay** | Integer (ms) | Maximum reconnection wait time |

### Device Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| **Local Address** | Integer | Master's DNP3 address (typically 1) |
| **Remote Address** | Integer | Outstation's DNP3 address |
| **Response Timeout** | Integer (ms) | Response wait time |
| **Integrity Scan Time** | Integer (ms) | Integrity scan period (all static data) |
| **Event Scan Time** | Integer (ms) | Event scan period (Class 1/2/3 events) |
| **Scan Type** | Enum | `PERIODIC` or `FIXED_DELAY` |
| **Unsolicited Events** | Boolean | Accept unsolicited responses |
| **Disable Unsolicited on Startup** | Boolean | Disable unsolicited on startup |
| **Startup Integrity** | Boolean | Perform integrity scan on startup |
| **Integrity on Event Overflow IIN** | Boolean | Integrity scan on event buffer overflow |

:::tip
**Integrity Scan** queries all static data and retrieves a complete copy of the outstation database. **Event Scan** fetches only data that has changed since the last scan. Both scans can be configured with separate periods.
:::

### Frame (Data Block) Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| **Type** | Enum | Data type (Binary Input, Analog Input, Counter, etc.) |
| **Start Address** | Integer | Starting point index |
| **Quantity** | Integer | Number of points |
| **Event Buffer Size** | Integer | Event buffer size |
| **Point Class** | String | Event class (Class 1, 2 or 3) |
| **Static Variation** | String | Static data format (e.g., Group30Var5 — 32-bit float) |
| **Event Variation** | String | Event data format |
| **Deadband** | Double | Analog event threshold (change amount) |

### Variable Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| **Start Address** | Integer | Point index |
| **Point Class** | String | Event class |
| **Static Variation** | String | Static data format |
| **Event Variation** | String | Event data format |
| **Deadband** | Double | Analog event threshold |

## DNP3 Master Configuration

### Step 1: Create a Connection

Select **DNP3** as the protocol and enter the target outstation's IP and port information.

### Step 2: Create a Device

- **Local Address:** Master address (typically 1)
- **Remote Address:** Outstation address (from device documentation)
- **Integrity Scan Time:** Recommended starting value 60000 ms (1 minute)
- **Event Scan Time:** Recommended starting value 5000 ms (5 seconds)

### Step 3: Define Frames and Variables

Create a Frame for each data group in the Outstation. Example:

- **Analog Input Frame:** Type = `Analog Input`, Start = 0, Quantity = 10
- **Binary Input Frame:** Type = `Binary Input`, Start = 0, Quantity = 32

Create a Variable for each point index within the Frame.

### Step 4: Start the Connection

Start the connection from the **Runtime Control Panel**. The connection status will show "Connected".

## DNP3 Slave (Outstation) Configuration

inSCADA can operate in DNP3 Outstation role, allowing external Masters to connect and read data. This mode is typically used to:

- Present data to upstream SCADA systems
- Transfer data to a control centre via DNP3
- Share data between different inSCADA instances

Select **DNP3 Slave** as the protocol for configuration. In the Device, Local Address and Remote Address roles are reversed — Local Address is the outstation's own address, Remote Address is the connecting Master's address.

:::caution
The DNP3 Slave port must be **open for inbound connections** in the firewall.
:::
