---
title: "Veri Aktarımı"
description: "Projeler arası değişken veri aktarımı ve istatistik hesaplama"
sidebar:
  order: 8
---

Data Transfer, bir projedeki değişken değerlerini başka bir projedeki değişkenlere periyodik olarak aktarır. Kaynak değişkenden istatistiksel hesaplama yaparak hedef değişkene yazmayı sağlar — sayaç diferansları, saatlik ortalamalar, çoklu saha toplama vb.

![Data Transfers](../../../../../assets/docs/dev-data-transfers.png)

## Data Transfer Alanları

| Alan | Tip | Zorunlu | Açıklama |
|------|-----|---------|----------|
| **name** | String (≤100) | Evet | Transfer adı |
| **dsc** | String (≤255) | Hayır | Açıklama |
| **period** | Integer (ms, ≥100) | Evet | Çalışma periyodu |
| **projectId** | String | Evet | Bağlı proje |

`period` ne kadar sıklıkla tetikleneceğini belirler — örneğin saatlik transfer için 3 600 000 ms.

## Transfer Detayları (DataTransferDetail)

Her Data Transfer bir veya daha fazla **detay satırı** içerir. Her detay bir kaynak-hedef eşleşmesidir:

| Alan | Açıklama |
|------|----------|
| **sourceVariableId** | Kaynak değişken |
| **targetVariableId** | Hedef değişken |
| **calcType** | İstatistiksel hesaplama tipi (`VariableStatCalculationType`) |
| **rangeType** | Zaman aralığı (`VariableStatRangeType`) |
| **threshold** | İstatistikler için opsiyonel filtre değeri |

Kaynak ve hedef farklı projelerde olabilir — bir projeden diğerine veri aktarmak için kullanılır.

## Hesaplama Tipleri

`VariableStatCalculationType` enum — 11 değer:

| Tip | Açıklama |
|-----|----------|
| **Min** | Aralıktaki minimum değer |
| **Max** | Aralıktaki maksimum değer |
| **Avg** | Aritmetik ortalama |
| **Sum** | Toplam |
| **Count** | Kayıt sayısı |
| **First Value** | Aralıktaki ilk kayıt |
| **Last Value** | Aralıktaki son kayıt |
| **Max Difference** | Aralıktaki en büyük ardışık fark |
| **Last First Difference** | Son değer − İlk değer (kümülatif sayaç farkı için ideal) |
| **Middle Value** | Aralığın ortasındaki değer (zamansal orta nokta) |
| **Median Value** | Medyan (sıralı ortanca) |

## Zaman Aralığı Tipleri

`VariableStatRangeType` enum — 10 değer (Current / Previous × 5 zaman ölçeği):

| Tip | Aralık |
|-----|--------|
| **Current Hour** | Saat başından şimdiye |
| **Previous Hour** | Bir önceki tam saat |
| **Current Day** | Gün başından şimdiye |
| **Previous Day** | Bir önceki tam gün |
| **Current Week** | Haftanın başından şimdiye |
| **Previous Week** | Bir önceki tam hafta |
| **Current Month** | Ay başından şimdiye |
| **Previous Month** | Bir önceki tam ay |
| **Current Year** | Yıl başından şimdiye |
| **Previous Year** | Bir önceki tam yıl |

`Previous ...` tipleri, aralık tamamlandığında tek seferlik hesaplama için idealdir — örneğin her sabah "önceki günün ortalaması" raporu.

## Kullanım Senaryoları

### Saatlik Enerji Tüketimi

Kümülatif sayaçtan saatlik tüketim:
- Kaynak: `Energy_kWh` (kümülatif)
- Hedef: `Hourly_Consumption`
- calcType: **Last First Difference** (son − ilk = saatlik tüketim)
- rangeType: **Previous Hour**
- period: 3 600 000 ms (saat başında tetiklenir)

### Günlük Ortalama Sıcaklık

- Kaynak: `Temperature_C`
- Hedef: `DailyAvg_Temperature`
- calcType: **Avg**
- rangeType: **Previous Day**
- period: 86 400 000 ms

### Projeler Arası Anlık Veri Kopyası

Birden fazla sahadaki güç değerlerini merkezi bir projeye aktarma:
- Kaynak (saha projesi): `Site1_Power_kW`
- Hedef (merkez proje): `Site1_Power_kW_Mirror`
- calcType: **Last Value**
- rangeType: **Current Hour** (canlı değer için kısa aralık)
- period: 10 000 ms

## Script ile Yönetim

```javascript
// Transfer'i zamanlayıcıya ekle
ins.scheduleDataTransfer("hourly_energy_calc");

// Projedeki tüm data transfer'leri zamanlayıcıya ekle
ins.scheduleDataTransfers();

// İptal et
ins.cancelDataTransfer("hourly_energy_calc");
ins.cancelDataTransfers();

// Durumu sorgula — "Scheduled" veya "Not Scheduled"
var status = ins.getDataTransferStatus("hourly_energy_calc");
```

`Scheduled` durumu "zamanlayıcıya bağlı" anlamına gelir — o anda çalıştığını değil.

Detaylı API: [Data Transfer API →](/docs/tr/jdk21/platform/scripts/server/datatransfer-api/) | [REST API Reference →](/docs/tr/jdk21/api/reference/) (Data Transfer Controller grubu)
