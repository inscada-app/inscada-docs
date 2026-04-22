---
title: "Report API"
description: "Klasik ve Jasper raporları zamanlama, dosyaya export ve e-posta ile dağıtma"
sidebar:
  order: 8
---

Report API; platformdaki rapor tanımlarını (klasik + Jasper) script'ten zamanlamak, iptal etmek, PDF/Excel dosyasına export etmek veya e-posta ile ulaştırmak için kullanılır.

## Rapor Durumu

### `ins.getReportStatus(reportName)`

`ReportStatus` enum döner — iki değer:

| Değer | Anlam |
| --- | --- |
| `"Scheduled"` | Zamanlayıcıya bağlı |
| `"Not Scheduled"` | Bağlı değil |

```javascript
if (ins.getReportStatus("daily_energy_report") == "Not Scheduled") {
    ins.scheduleReport("daily_energy_report");
}
```

## Zamanlama / İptal

### `ins.scheduleReport(reportName)`

Tek bir raporu zamanlayıcıya ekler — rapor tanımındaki periyoda göre çalışır.

```javascript
ins.scheduleReport("daily_energy_report");
```

### `ins.cancelReport(reportName)`

Raporu zamanlayıcıdan çıkarır — kalan tetiklemeler iptal; o anda üretilmekte olan bir rapor doğal olarak tamamlanır.

```javascript
ins.cancelReport("daily_energy_report");
```

### `ins.scheduleReports()` / `ins.cancelReports()`

Projedeki **tüm** rapor tanımlarını bir seferde zamanla ya da iptal et.

```javascript
ins.scheduleReports();
// ...
ins.cancelReports();
```

## E-posta ile Dağıtım

### `ins.mailReport(reportName, startDate, endDate)`

**Klasik** (Jasper olmayan) raporu verilen tarih aralığı için üretir ve rapor tanımındaki alıcılara e-posta ile gönderir.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 86400000);   // son 24 saat
ins.mailReport("daily_energy_report", start, end);
```

### `ins.mailJasperReport(reportName, params, usernames, subject, content)`

Jasper raporunu **PDF** ek olarak verilen kullanıcılara e-postalar.

| Parametre | Tür | Açıklama |
| --- | --- | --- |
| `reportName` | `String` | Jasper rapor tanım adı |
| `params` | `Map<String, Object>` | Rapora geçirilecek parametreler |
| `usernames` | `String[]` | Alıcı kullanıcı adları (e-posta profilden) |
| `subject` | `String` | E-posta konusu |
| `content` | `String` | E-posta gövdesi |

```javascript
var params = {
    "START_DATE": ins.getDate(ins.now().getTime() - 86400000),
    "END_DATE":   ins.now(),
    "PROJECT_ID": 153
};

ins.mailJasperReport(
    "energy_report",
    params,
    ["manager", "operator"],
    "Günlük Enerji Raporu",
    "Ekteki PDF rapor otomatik oluşturulmuştur."
);
```

### `ins.mailJasperExcelReport(reportName, params, usernames, subject, content)`

Aynı imza — ama PDF yerine **Excel** ek olarak gönderir.

```javascript
ins.mailJasperExcelReport(
    "energy_report",
    params,
    ["manager"],
    "Günlük Enerji Raporu (Excel)",
    "Ekteki Excel rapor otomatik oluşturulmuştur."
);
```

## Dosyaya Export

### `ins.exportJasperPdfToFile(reportName, filePath)`

Jasper raporunu **PDF** olarak platformun dosya sistemine yazar.

```javascript
ins.exportJasperPdfToFile("energy_report", "reports/daily_report.pdf");
```

### `ins.exportJasperExcelToFile(reportName, filePath)`

Jasper raporunu **Excel** olarak yazar.

```javascript
ins.exportJasperExcelToFile("energy_report", "reports/daily_report.xlsx");
```

:::note
`filePath`, platformun yönetilen dosya sistemine görelidir (aynı `ins.readFile` / `ins.writeToFile` ile kullandığın ağaç). Export edilen dosyayı sonra `ins.readFileAsBytes()` ile okuyup başka bir kanala taşıyabilirsin.
:::

## Örnek: Günlük Otomatik Rapor

Her gün 08:00'de önceki günün raporunu PDF dosyasına yazar + e-postalar.

```javascript
// Schedule Type: Daily, Time: 08:00
function main() {
    var now = ins.now();
    var y   = ins.getDate(now.getTime() - 86400000);

    var yyyy = 1900 + y.getYear();
    var mm   = ins.leftPad(String(y.getMonth() + 1), 2, "0");
    var dd   = ins.leftPad(String(y.getDate()),     2, "0");
    var file = "reports/energy_" + yyyy + mm + dd + ".pdf";

    ins.exportJasperPdfToFile("energy_report", file);
    ins.mailReport("energy_report", y, now);
    ins.writeLog("info", "Report", "Günlük rapor oluşturuldu: " + file);
}
main();
```
