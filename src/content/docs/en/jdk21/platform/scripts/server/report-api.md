---
title: "Report API"
description: "Schedule classic and Jasper reports, export them to files, and deliver them by email"
sidebar:
  order: 8
---

Report API lets a script schedule and cancel the report definitions on the platform (classic + Jasper), export them to PDF / Excel files, or send them by email.

## Report Status

### `ins.getReportStatus(reportName)`

Returns a `ReportStatus` enum — two values:

| Value | Meaning |
| --- | --- |
| `"Scheduled"` | Attached to the scheduler |
| `"Not Scheduled"` | Not attached |

```javascript
if (ins.getReportStatus("daily_energy_report") == "Not Scheduled") {
    ins.scheduleReport("daily_energy_report");
}
```

## Schedule / Cancel

### `ins.scheduleReport(reportName)`

Adds a single report to the scheduler — it runs at the period in the report definition.

```javascript
ins.scheduleReport("daily_energy_report");
```

### `ins.cancelReport(reportName)`

Removes the report from the scheduler — remaining triggers are cancelled; a report currently being generated finishes normally.

```javascript
ins.cancelReport("daily_energy_report");
```

### `ins.scheduleReports()` / `ins.cancelReports()`

Schedule or cancel **every** report definition in the project at once.

```javascript
ins.scheduleReports();
// ...
ins.cancelReports();
```

## Email Delivery

### `ins.mailReport(reportName, startDate, endDate)`

Generates a **classic** (non-Jasper) report for the given range and emails it to the recipients defined in the report.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 86400000);   // last 24 hours
ins.mailReport("daily_energy_report", start, end);
```

### `ins.mailJasperReport(reportName, params, usernames, subject, content)`

Emails a Jasper report as a **PDF** attachment to the given users.

| Parameter | Type | Description |
| --- | --- | --- |
| `reportName` | `String` | Jasper report definition name |
| `params` | `Map<String, Object>` | Parameters passed to the report |
| `usernames` | `String[]` | Recipient platform usernames (emails resolved from profiles) |
| `subject` | `String` | Email subject |
| `content` | `String` | Email body |

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
    "Daily Energy Report",
    "The attached PDF report is generated automatically."
);
```

### `ins.mailJasperExcelReport(reportName, params, usernames, subject, content)`

Same signature — but attaches an **Excel** file instead of PDF.

```javascript
ins.mailJasperExcelReport(
    "energy_report",
    params,
    ["manager"],
    "Daily Energy Report (Excel)",
    "The attached Excel report is generated automatically."
);
```

## Export to File

### `ins.exportJasperPdfToFile(reportName, filePath)`

Writes a Jasper report as **PDF** to the platform's file system.

```javascript
ins.exportJasperPdfToFile("energy_report", "reports/daily_report.pdf");
```

### `ins.exportJasperExcelToFile(reportName, filePath)`

Writes a Jasper report as **Excel**.

```javascript
ins.exportJasperExcelToFile("energy_report", "reports/daily_report.xlsx");
```

:::note
`filePath` is relative to the platform's managed file system (the same tree you use with `ins.readFile` / `ins.writeToFile`). You can pick up the exported file later with `ins.readFileAsBytes()` to forward it through another channel.
:::

## Example: Daily Automated Report

Runs at 08:00 every day, writes yesterday's report to a PDF file and emails it.

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
    ins.writeLog("info", "Report", "Daily report generated: " + file);
}
main();
```
