---
title: "OPC UA Client"
description: "inSCADA'da OPC UA Client bağlantı yapılandırması"
sidebar:
  order: 1
  label: "OPC UA Client"
---

Client modunda inSCADA, OPC UA sunucularına (PLC, DCS, gateway, başka bir SCADA vb.) bağlanarak node değerlerini okur ve yazar.

## Adım 1: Connection Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Protocol** | OPC UA | Protokol seçimi |
| **IP Address** | 192.168.1.100 | OPC UA sunucu IP adresi |
| **Port** | 4840 | OPC UA portu (varsayılan: 4840) |
| **Server Name** | (boş) | Sunucu discovery adı (opsiyonel) |
| **Timeout** | 5000 ms | İstek timeout süresi |
| **Security Mode** | SignAndEncrypt | `None`, `Sign` veya `SignAndEncrypt` |
| **Security Policy** | Basic256Sha256 | `None`, `Basic128Rsa15`, `Basic256`, `Basic256Sha256`, `Aes128_Sha256_RsaOaep` |
| **Anonymous Auth** | false | Anonim kimlik doğrulama |
| **Username/Password Auth** | true | Kullanıcı adı/şifre ile kimlik doğrulama |
| **Username** | operator | Kullanıcı adı |
| **Password** | •••• | Şifre |
| **Encoding Type** | Binary | `Binary`, `Xml` veya `Json` |
| **HTTPS Enabled** | false | HTTPS transport kullan |
| **HTTPS Port** | 443 | HTTPS portu (HTTPS etkinse) |

### Sertifika Yönetimi

Security Mode `Sign` veya `SignAndEncrypt` seçildiğinde, inSCADA otomatik olarak bir client sertifikası oluşturur. İlk bağlantıda OPC UA sunucusu bu sertifikayı reddedebilir — sunucu tarafında sertifikanın **trust** edilmesi gerekir.

Sertifika dosyaları: `<user_home>/opc-ua/client/security/` dizininde saklanır.

:::tip
İlk bağlantıda "BadSecurityChecksFailed" hatası alıyorsanız, OPC UA sunucusunun yönetim panelinden inSCADA sertifikasını güvenilir (trusted) listesine ekleyin.
:::

## Adım 2: Device Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Base Path** | `PLC_1` | Node ağacında temel yol (gruplama için) |
| **Scan Time** | 1000 ms | Tarama periyodu |
| **Scan Type** | PERIODIC | `PERIODIC` veya `FIXED_DELAY` |

## Adım 3: Frame Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Path** | `Channel1.Device1` | Node ağacında frame yolu |

Frame, OPC UA adres alanında bir dallanma noktasını temsil eder. İlgili node'ları gruplamak için kullanılır.

## Adım 4: Variable Oluşturma

| Parametre | Örnek | Açıklama |
|-----------|-------|----------|
| **Namespace Index** | 2 | Node'un namespace indeksi |
| **Identifier Type** | String | `Numeric` veya `String` |
| **Identifier** | `PLC1.Temperature` | Node identifier değeri |
| **Type** | Float | Veri tipi |
| **Is Array** | false | Dizi değişken mi? |
| **First Dimension Index** | 0 | Dizi birinci boyut indeksi |
| **Second Dimension Index** | 0 | Dizi ikinci boyut indeksi |

### Node ID Örnekleri

| Tanım | Namespace | Identifier Type | Identifier | Node ID |
|-------|:---------:|:--------------:|------------|---------|
| Sıcaklık | 2 | String | `PLC1.Temp` | `ns=2; s="PLC1.Temp"` |
| Basınç | 2 | Numeric | `1001` | `ns=2; i=1001` |
| Durum | 3 | String | `Sensor1.Status` | `ns=3; s="Sensor1.Status"` |

### Dizi (Array) Değişkenler

OPC UA sunucusundaki bir node dizi değer içeriyorsa, `Is Array = true` olarak işaretleyin ve ilgili boyut indekslerini girin. Bu özellikle çok boyutlu veri yapıları (ör. tarihsel veri blokları, parametre setleri) için kullanılır.

## Adım 5: Bağlantıyı Başlatma

**Runtime Control Panel**'den bağlantıyı başlatın. inSCADA:
1. OPC UA endpoint'ini keşfeder
2. Güvenlik ayarlarına uygun endpoint seçer
3. Sertifika doğrulaması yapar
4. Session oluşturur
5. Periyodik okumaya başlar

Bağlantı durumu "Connected" olarak görünecektir.

## Browse (Keşif)

inSCADA, OPC UA sunucusunun adres alanını **browse** ederek mevcut node'ları keşfedebilir. Bu özellik, variable tanımlarken doğru Node ID bilgilerine ulaşmanızı kolaylaştırır.
