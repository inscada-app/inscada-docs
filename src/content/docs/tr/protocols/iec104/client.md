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
**Use IOA Addresses = false** durumunda basit `Read Address` / `Write Address` kullanılır. **true** durumunda 3 byte'lık IOA adresleri ayrı ayrı girilir (IOA Field Length = 3 byte ise).
:::

### Read ve Write Adresi Kullanım Yaklaşımı

inSCADA'da bir IEC 104 variable'ına hem **Read Address** hem de **Write Address** girilebilir. Bu iki adresin doğru kullanımı, verimli bir yapılandırma için kritiktir.

**Temel kural:** Bir variable'a yalnızca Write Address girerseniz, yazılan değeri inSCADA ekranlarında **göremezsiniz** — çünkü okuma adresi tanımlı değildir.

#### En İyi Yaklaşım: Aynı IOA Adresini Kullanın

IEC 104 standardında, aynı veri noktası için okuma (monitoring) ve yazma (control) farklı ASDU tipleri ile yapılır. Ancak **IOA adresi aynı kalabilir** — çünkü ASDU tipi Frame'in tipinden belirlenir. inSCADA hangi ASDU tipini kullanacağını zaten bilir:

| İşlem | ASDU Tipi | Tip Grubu | Yön |
|-------|-----------|-----------|-----|
| **Okuma** | M_SP_NA_1 (TI 1) | Single Point Information | Monitoring |
| **Okuma (zaman damgalı)** | M_SP_TB_1 (TI 30) | Single Point Information | Monitoring |
| **Yazma** | C_SC_NA_1 (TI 45) | Single Command | Control |
| **Yazma (zaman damgalı)** | C_SC_TA_1 (TI 58) | Single Command | Control |

Dikkat edin: Bunların hepsi aynı **Single Point** veri grubuna aittir. Fark yalnızca monitoring (M_) veya control (C_) olması ve zaman damgası içerip içermemesidir. Dolayısıyla **aynı IOA adresini hem Read hem Write olarak girebilirsiniz:**

```
Variable: "Kesici_1"
├── Read Address:  100    ← M_SP_NA_1 olarak okunur (Frame tipi belirler)
└── Write Address: 100    ← C_SC_NA_1 olarak yazılır (Frame tipi belirler)
```

inSCADA, variable'ın bağlı olduğu Frame tipine bakarak otomatik olarak doğru ASDU tipini seçer:
- Frame tipi **Single Point Information** → okuma için M_SP_NA_1/TB_1, yazma için C_SC_NA_1/TA_1
- Frame tipi **Measured Value, Short Float** → okuma için M_ME_NC_1/TF_1, yazma için C_SE_NC_1

**Bu sayede:**
- Aynı IOA adresi ile tek bir variable tanımı yeterlidir
- RTU/PLC tarafında da aynı adres hem monitoring hem control için kullanılabilir
- Okuma ve yazma için **ayrı variable oluşturmanıza gerek kalmaz**
- Ekranlarda değişkenin güncel değerini görebilir, aynı değişken üzerinden kontrol komutu gönderebilirsiniz

**Örnek — Kesici Kontrolü:**

| Variable | Read Address | Write Address | Okuma ASDU | Yazma ASDU |
|----------|:-----------:|:------------:|:----------:|:----------:|
| Kesici_1 | 100 | 100 | M_SP_NA_1 | C_SC_NA_1 |
| Sıcaklık_Setpoint | 200 | 200 | M_ME_NC_1 | C_SE_NC_1 |

:::caution[Önemli Tavsiye]
IEC 104 kullandığınız uygulamalarınızda bu adresleme yaklaşımını standart olarak benimsemenizi önemle tavsiye ederiz. RTU/PLC tarafında aynı IOA adresinin hem monitoring hem control için yapılandırılması, inSCADA tarafındaki variable sayısını yarıya indirir ve proje yönetimini önemli ölçüde basitleştirir.
:::

## Adım 5: Bağlantıyı Başlatma

**Runtime Control Panel**'den bağlantıyı başlatın. inSCADA otomatik olarak:
1. TCP bağlantısı kurar
2. STARTDT (Start Data Transfer) gönderir
3. Background scan başlatır (yapılandırılmışsa)
4. Spontaneous event'leri dinlemeye başlar

Bağlantı durumu "Connected" olarak görünecektir.
