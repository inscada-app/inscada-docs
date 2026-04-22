---
title: "System API"
description: "Shutdown / restart, OS komutu çalıştırma, sistem tarihi ayarlama ve sistem istek kuyruğu"
sidebar:
  order: 12
---

System API, platformun çalıştığı sunucu üzerinde OS seviyesinde işlem yapar: platformu kapatıp yeniden başlatır, sistem saatini ayarlar, keyfi bir komut çalıştırır ve bekleyen sistem isteklerini yönetir.

:::caution
Bu API çağrılarının hepsi **`EXEC_SYSTEM_COMMAND`** yetkisi gerektirir. Yanlış kullanım sunucuyu durdurur / saati bozar / arbitrary code execution'a dönüşebilir. Her çağrı proje log'una audit olarak düşer.
:::

## `ins.shutdown()`

Platform sürecini kapatır.

```javascript
ins.shutdown();
```

## `ins.restart()`

Platformu yeniden başlatır.

```javascript
var h = ins.now().getHours();
if (h >= 2 && h <= 4) {
    ins.writeLog("warn", "System", "Planlı yeniden başlatma");
    ins.restart();
}
```

## `ins.setDateTime(ms, dateCmdFormat)`

Sunucu sistem saatini `ms` (epoch) değerine ayarlar.

| Parametre | Açıklama |
| --- | --- |
| `ms` | Hedef zaman (epoch milisaniye) |
| `dateCmdFormat` | **Yalnızca Windows'ta** kullanılan Java `SimpleDateFormat` pattern'i (örn. `"MM-dd-yyyy"`) — pattern `cmd /c date <değer>`'e geçirilir, saat ayrıca `HH:mm:ss` ile `time` komutuna iletilir. **Linux'ta bu parametre yok sayılır** — dahili olarak sabit `yyyy-MM-dd HH:mm:ss` biçimi + `date -s` kullanılır |

```javascript
// Windows
ins.setDateTime(Date.now(), "MM-dd-yyyy");

// Linux — format görmezden gelinir, boş bırakabilirsin
ins.setDateTime(Date.now(), "");
```

## `ins.exec(command)` — İki Overload

Sunucuda OS komutu çalıştırır. Dönüş değeri komut çıktısı DEĞİL, **exit code** (int) — `0` başarı.

### `ins.exec(String[] command)` *(önerilen)*

Argüman dizisi olarak verilir — shell parse etmesi yoktur, injection riski düşük.

```javascript
var rc = ins.exec(["df", "-h", "/"]);
if (rc != 0) {
    ins.writeLog("error", "System", "df komutu başarısız — exit " + rc);
}
```

### `ins.exec(String commandLine)`

Tek string olarak verilir, içsel olarak boşluklara göre parse edilir (yalın yaklaşım, tırnak içi argümanlara duyarlı değil).

```javascript
ins.exec("df -h /");
```

:::caution
Komut çıktısını almak ISTIYORSAN `exec` yetmez. Komutu `> /opt/inscada/tmp/out.txt` şeklinde dosyaya yönlendir, sonra `ins.readFile("tmp/out.txt")` ile oku. `exec` sadece exit code döner.
:::

:::caution
Kullanıcı girdisini **asla** doğrudan `exec`'e geçirme. İnjekte edilirse attacker OS komutu çalıştırabilir. Önceden doğrula / whitelist uygula.
:::

## Sistem İstek Kuyruğu

Platform içinde "sistem isteği" (kullanıcıdan onay bekleyen shutdown / restart gibi) kuyruklanabilir. API bu kuyruğa script'ten erişim sağlar.

### `ins.getSystemRequests()`

Bekleyen sistem isteklerini döner — `Collection<SystemRequestDto>`.

```javascript
var reqs = ins.getSystemRequests();
reqs.forEach(function(r) {
    ins.consoleLog(r.getType() + " — " + r.getRequestDate());
});
```

### `ins.deleteSystemRequest(systemRequest)`

Bir isteği kuyruktan siler.

```javascript
var reqs = ins.getSystemRequests();
reqs.forEach(function(r) {
    if (r.getType() == "RESTART") ins.deleteSystemRequest(r);
});
```

### `SystemRequestDto` Alanları

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getType()` | `String` | İstek tipi (örn. `"SHUTDOWN"`, `"RESTART"`) |
| `getRequester()` | `Map<String, Object>` | İsteği yapan kullanıcı bilgileri |
| `getRequestDate()` | `Date` | Oluşturulma zamanı |

## Örnek: Düşük Disk Alanında Planlı Restart

```javascript
function main() {
    var rc = ins.exec(["sh", "-c", "df -h / | awk 'NR==2 {print $5}' | tr -d '%' > /opt/inscada/tmp/disk.txt"]);
    if (rc != 0) return;

    var used = parseInt(ins.readFile("tmp/disk.txt").trim(), 10);
    if (used > 95) {
        ins.sendMail(["ops"], "Disk %" + used, "Kritik disk doluluk — bakım penceresinde restart planlanıyor");
    }
}
main();
```
