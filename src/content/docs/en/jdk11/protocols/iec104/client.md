---
title: "IEC 104 Client (Master)"
description: "inSCADA'da IEC 60870-5-104 Client (Master) bağlantı yapılandırması"
sidebar:
  order: 1
  label: "IEC 104 Client"
---

Client (Master) modunda inSCADA, saha istasyonlarına (RTU, IED, koruma rölesi vb.) bağlanarak veri okur ve kontrol komutları gönderir.

## Adım 1: Connection Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | IEC 60870-5-104 | Protokol seçimi |
| **IP Address** | 192.168.1.50 | Hedef slave istasyon IP adresi |
| **Port** | 2404 | Hedef port (varsayılan: 2404) |
| **COT Field Length** | 2 | Cause of Transmission alan uzunluğu (1-2 byte) |
| **Common Address Field Length** | 2 | CASDU alan uzunluğu (1-2 byte) |
| **IOA Field Length** | 3 | IOA alan uzunluğu (1-3 byte) |
| **Originator Address** | 0 | Master originator adresi |
| **t1** | 15 sn | Yanıt bekleme timeout'u |
| **t2** | 10 sn | S-format onay timeout'u |
| **t3** | 20 sn | Test frame timeout'u |
| **k** | 12 | Maks. onaylanmamış I-frame |
| **w** | 8 | Onay penceresi |
| **Is With Timestamps** | true | Zaman damgalı ASDU'ları kabul et |
| **Background Scan Period** | 60000 ms | Background scan periyodu |
| **Spontaneous Duplicates** | 0 | Spontaneous mesaj tekrar sayısı |
| **Use System Timezone** | false | Sistem zaman dilimini kullan |

:::tip
**t1, t2, t3, k, w** parametreleri her iki tarafta da (Master ve Slave) aynı değerlere ayarlanmalıdır. Uyumsuz değerler bağlantı kopmalarına neden olabilir.
:::

## Adım 2: Device Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Common Address** | 1 | CASDU — slave istasyon adresi |
| **Control Point Offset** | 0 | Kontrol komutu adres ofseti |
| **Scan Time** | 1000 ms | Tarama periyodu |
| **Scan Type** | PERIODIC | `PERIODIC` veya `FIXED_DELAY` |

## Adım 3: Frame Oluşturma

Her ASDU tipi için bir Frame tanımlayın:

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Type** | Measured Value, Short Float | ASDU tipi |

### Tipik Frame Örnekleri

| Frame Adı | Type | Kullanım |
|-----------|------|----------|
| Dijital Giriş | Single Point Information | Kesici/ayırıcı durumları |
| Çift Dijital | Double Point Information | Kesici pozisyonları (açık/kapalı/geçiş/belirsiz) |
| Analog Ölçüm | Measured Value, Short Float | Akım, gerilim, güç ölçümleri |
| Normalize Ölçüm | Measured Value, Normalized | ±32767 aralığında tam sayı ölçümler |
| Ölçekli Ölçüm | Measured Value, Scaled | Ölçeklenmiş tam sayı ölçümler |

## Adım 4: Variable Oluşturma

IEC 104 değişkenleri IOA (Information Object Address) ile tanımlanır:

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Use IOA Addresses** | true | IOA adresleme modu |
| **Read Address** | 100 | Okuma adresi (basit mod) |
| **Write Address** | 100 | Yazma adresi (basit mod) |
| **Read IOA Address 1** | 0 | IOA okuma adresi byte 1 |
| **Read IOA Address 2** | 0 | IOA okuma adresi byte 2 |
| **Read IOA Address 3** | 100 | IOA okuma adresi byte 3 |
| **Write IOA Address 1** | 0 | IOA yazma adresi byte 1 |
| **Write IOA Address 2** | 0 | IOA yazma adresi byte 2 |
| **Write IOA Address 3** | 100 | IOA yazma adresi byte 3 |

:::note
When **Use IOA Addresses = false**, simple `Read Address` / `Write Address` fields are used. When **true**, 3-byte IOA addresses are entered separately (when IOA Field Length = 3 bytes).
:::

### Read and Write Address Usage Pattern

In inSCADA, an IEC 104 variable can have both a **Read Address** and a **Write Address**. Correct usage of these two addresses is critical for efficient configuration.

**Basic rule:** If you only enter a Write Address for a variable, you **cannot see** the written value on inSCADA screens — because no read address is defined.

#### Best Practice: Use the Same IOA Address

In the IEC 104 standard, reading (monitoring) and writing (control) for the same data point use different ASDU types. However, the **IOA address can remain the same** — because the ASDU type is determined by the Frame type. inSCADA automatically knows which ASDU type to use:

| Operation | ASDU Type | Type Group | Direction |
|-----------|-----------|------------|-----------|
| **Read** | M_SP_NA_1 (TI 1) | Single Point Information | Monitoring |
| **Read (timestamped)** | M_SP_TB_1 (TI 30) | Single Point Information | Monitoring |
| **Write** | C_SC_NA_1 (TI 45) | Single Command | Control |
| **Write (timestamped)** | C_SC_TA_1 (TI 58) | Single Command | Control |

Notice: These all belong to the same **Single Point** data group. The only difference is whether it is monitoring (M_) or control (C_) and whether it includes a timestamp. Therefore, **you can enter the same IOA address for both Read and Write:**

```
Variable: "Breaker_1"
├── Read Address:  100    ← Read as M_SP_NA_1 (Frame type determines this)
└── Write Address: 100    ← Written as C_SC_NA_1 (Frame type determines this)
```

inSCADA automatically selects the correct ASDU type by looking at the Frame type the variable belongs to:
- Frame type **Single Point Information** → M_SP_NA_1/TB_1 for reading, C_SC_NA_1/TA_1 for writing
- Frame type **Measured Value, Short Float** → M_ME_NC_1/TF_1 for reading, C_SE_NC_1 for writing

**This means:**
- A single variable definition with the same IOA address is sufficient
- The same address can be used for both monitoring and control on the RTU/PLC side
- **No need to create separate variables** for reading and writing
- You can see the current value on screens and send control commands from the same variable

**Example — Breaker Control:**

| Variable | Read Address | Write Address | Read ASDU | Write ASDU |
|----------|:-----------:|:------------:|:---------:|:----------:|
| Breaker_1 | 100 | 100 | M_SP_NA_1 | C_SC_NA_1 |
| Temp_Setpoint | 200 | 200 | M_ME_NC_1 | C_SE_NC_1 |

:::caution[Important Recommendation]
We strongly recommend adopting this addressing approach as a standard practice in your IEC 104 applications. Configuring the same IOA address for both monitoring and control on the RTU/PLC side halves the number of variables in inSCADA and significantly simplifies project management.
:::

## Step 5: Start the Connection

Start the connection from the **Runtime Control Panel**. inSCADA will automatically:
1. Establish the TCP connection
2. Send STARTDT (Start Data Transfer)
3. Start background scan (if configured)
4. Begin listening for spontaneous events

The connection status will show "Connected".
