---
title: "Grafik & Tablolar"
description: "Chart, Peity (sparkline) ve Datatable animation tipleri"
sidebar:
  order: 3
---

## Chart (Grafik)

SVG ekran içine interaktif grafik bileşeni yerleştirir. Tarihsel veri veya anlık değer serilerini görselleştirir.

| Alan | Değer |
|------|-------|
| **Type** | Chart |
| **DOM ID** | SVG foreignObject veya group |
| **Expression Type** | Collection veya Expression |
| **Props** | Grafik yapılandırması (tip, renkler, eksenler) |

### Grafik Tipleri

| Tip | Kullanım |
|-----|----------|
| **Line** | Zaman serisi, trend takibi |
| **Bar** | Karşılaştırma, periyodik özet |
| **Area** | Kümülatif değerler |
| **Pie** | Dağılım gösterimi |

### Örnek: Script ile Chart Verisi

Animation script'i ile tarihsel veri çekip grafiğe aktarma:

```javascript
var endMs = ins.now().getTime();
var startMs = endMs - (60 * 60 * 1000); // 1 saat

var logs = ins.getLoggedVariableValuesByPage(
    Java.to(['ActivePower_kW'], 'java.lang.String[]'),
    ins.getDate(startMs), ins.getDate(endMs), 0, 30
);

var labels = [];
var values = [];
for (var i = logs.length - 1; i >= 0; i--) {
    labels.push(logs[i].dttm);
    values.push(logs[i].value);
}

return {
    type: 'line',
    labels: labels,
    dataset: {
        0: {
            name: 'Active Power (kW)',
            data: values,
            color: '#1CA1C1',
            fill: true
        }
    }
};
```

## Peity (Sparkline Mini Grafik)

Küçük inline sparkline grafik. Metin yanında kompakt trend gösterimi için kullanılır. Fazla yer kaplamadan değerin yönünü gösterir.

| Alan | Değer |
|------|-------|
| **Type** | Peity |
| **DOM ID** | SVG foreignObject |
| **Expression Type** | Collection |

Peity tipleri: line, bar, pie, donut

```xml
<!-- SVG içinde küçük sparkline alanı -->
<foreignObject id="power_sparkline" x="200" y="10" width="80" height="30"/>
```

Son N değeri mini grafik olarak gösterir — değerin artış/düşüş trendini hızlıca görselleştirir.

## Datatable (Tablo)

SVG ekran içine interaktif tablo bileşeni yerleştirir. Birden fazla değişkeni tablo formatında listelemek için kullanılır.

| Alan | Değer |
|------|-------|
| **Type** | Datatable |
| **DOM ID** | SVG foreignObject |
| **Expression Type** | Collection veya Expression |
| **Props** | Kolon tanımları, sıralama, filtreleme |

Kullanım senaryoları:
- Alarm listesi tablosu
- Değişken izleme tablosu (isim, değer, birim, zaman)
- Tarihsel veri tablosu
