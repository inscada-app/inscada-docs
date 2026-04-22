---
title: "Server-Side Script Engine"
description: "ins.* — GraalJS tabanlı, Nashorn uyumluluk modlu server-side script motoru"
sidebar:
  order: 0
  label: "Genel Bakış"
---

Server-side script'ler inSCADA sunucusunda çalışır ve `ins` global objesi üzerinden platformun tüm yeteneklerine erişir. Bu sayfa engine, sandbox, zamanlama ve yaşam döngüsünü özetler; alt API'lerin detayı ayrı sayfalardadır.

## Engine

JDK21 sürümünde script motoru **GraalJS** (GraalVM JavaScript) ile değiştirildi. Mevcut JDK11 script'lerinin kırılmadan çalışabilmesi için **Nashorn compatibility mode** aktif edildi (`js.nashorn-compat: true`). Pratik sonuç:

- JDK11'de yazılmış script'leriniz (yalnızca `var`, `function`, ES5) değişiklik yapmadan çalışır.
- İsterseniz **modern JS** kullanabilirsiniz: `let` / `const`, arrow fonksiyonları, template literal, destructuring, class, spread / rest operatörleri, `async` / `await`.
- İki stili aynı script içinde karıştırmak güvenlidir.

### Desteklenen JavaScript Özellikleri

| Özellik | Durum |
| --- | --- |
| `var`, `function`, ES5 standardı | ✓ |
| `let`, `const` | ✓ |
| Arrow fonksiyon (`=>`) | ✓ |
| Template literal (backtick) | ✓ |
| Destructuring | ✓ |
| `class` / `extends` | ✓ |
| Spread / rest (`...`) | ✓ |
| `async` / `await` | ✓ |
| `eval(...)` | ✗ (sandbox kapalı) |
| `with (...)` | ✗ (sandbox kapalı) |

## Sandbox Kısıtları

Script'lerin güvenliği Java host access üzerinden kontrol edilir. Script içinden yalnızca `@HostAccess.Export` ile işaretli Java metodları çağrılabilir — `ins.*` bu metodların script'e açılmış halidir.

### Yasaklar

| Yasak | Neden |
| --- | --- |
| Thread oluşturma | Script'ler paylaşımlı thread havuzunda çalışır |
| Doğrudan dosya sistemi erişimi | Yalnızca `ins.readFile()` / `ins.writeToFile()` üzerinden |
| Native code (JNI / JNA) | — |
| Environment variable erişimi | — |
| Polyglot (diğer diller) | — |
| `eval`, `with` | — |

`exec`, `shutdown`, `restart` gibi sistem komutları yalnızca `ins.*` üzerinden ve proje log'una düşerek çalışır.

### Kaynak Limitleri

| Limit | Default | Config anahtarı |
| --- | --- | --- |
| Statement sayısı | 100 000 | `ins.script.maxStatementCount` |
| Yürütme süresi | 60 saniye | `ins.script.execution-timeout` |

Limiti aşan script iptal edilir ve `ScriptException` ile sonlanır; hata log'a düşer, platform etkilenmez.

## Global Binding'ler

Script context'ine otomatik enjekte edilen objeler:

| Global | Tür | Açıklama |
| --- | --- | --- |
| `ins` | `InscadaApi` | Platformun tüm alt API'lerinin birleşimi (aşağıda liste) |
| `user` | `CurrentUserBinding` | Aktif kullanıcı bilgisi — `id`, `name`, `roles`, `permissions`, `activeSpace`, `remoteAddress`, `spaces`, `menus` |
| Custom bindings | opsiyonel | Script tanımı içinde `bindings` alanında verilen ek objeler |

Nashorn'un meta global'leri (`quit`, `exit`, `print`, `load`, `loadWithNewGlobal`, `$ARG`, `$ENV`, `$EXEC`, `$OPTIONS`, `$OUT`, `$ERR`, `$EXIT`) güvenlik için no-op yapılmıştır — çağırıldıklarında hiçbir şey yapmazlar.

## Scheduled Scripts

Bağımsız otomasyon görevleri **Development → Scripts** menüsünden tanımlanır.

### Zamanlama Tipleri

| Tip | Parametreler | Açıklama |
| --- | --- | --- |
| **Periodic** | Period (ms), Offset (ms) | Sabit aralıkla tekrar |
| **Daily** | Saat:Dakika | Her gün belirli bir saatte |
| **Once** | Gecikme (ms) | Tek seferlik çalıştırma |
| **None** | — | Otomatik tetikleme yok, `ins.executeScript()` / REST ile manuel |

### Script Tanım Alanları

| Alan | Açıklama |
| --- | --- |
| **Name** | Benzersiz script adı |
| **Code** | JavaScript kaynak kodu |
| **Schedule Type** | `Periodic` / `Daily` / `Once` / `None` |
| **Period / Time** | Zamanlama parametresi |
| **Log** | Çalıştırma log'unu kaydet |
| **Compile** | Derleme cache aktif (önerilen: açık) |
| **Bindings** | İsteğe bağlı ek context bağlamaları |

### REST Üzerinde Script

```json
{
  "id": 159,
  "name": "Chart_ActiveReactivePower",
  "projectId": 153,
  "type": "None",
  "log": false,
  "compile": true,
  "owner": "inscada",
  "code": "function main() { /* ... */ } main();"
}
```

## Yaşam Döngüsü

1. **Derleme** — Kod SHA-256 ile hash'lenir, Caffeine cache'e yazılır (default 2000 entry, 60 dk idle TTL). Aynı kod tekrar çalıştığında yeniden derlenmez.
2. **Yürütme** — `inSCADA-script-executor-*` daemon thread havuzunda çalışır (core = CPU sayısı, max = 8 × CPU, kuyruk = 2048).
3. **Sonuç** — Return değeri GraalVM `Value`'den Java objesine açılır — `Map<String, Object>` / `List` / primitive olarak döner.
4. **Hata / Timeout** — Timeout aşılırsa `Future.cancel(true)` ile iptal edilir; hata proje log'una `ScriptException` olarak düşer.

## Örnek

Aynı amaç için hem ES5 hem modern JS ile yazılmış örnekler — ikisi de geçerlidir:

```javascript
// ES5 stil (JDK11 script'leriyle uyumlu)
function main() {
    var active = ins.getVariableValue("ActivePower_kW");
    var reactive = ins.getVariableValue("ReactivePower_kVAr");
    var s = Math.sqrt(active.value * active.value + reactive.value * reactive.value);
    ins.setVariableValue("ApparentPower_kVA", { value: s });
    ins.writeLog("INFO", "PowerCalc", "S = " + s.toFixed(2) + " kVA");
}
main();
```

```javascript
// Modern JS stil (JDK21'de isteğe bağlı)
function main() {
    const active = ins.getVariableValue("ActivePower_kW");
    const reactive = ins.getVariableValue("ReactivePower_kVAr");
    const s = Math.sqrt(active.value ** 2 + reactive.value ** 2);
    ins.setVariableValue("ApparentPower_kVA", { value: s });
    ins.writeLog("INFO", "PowerCalc", `S = ${s.toFixed(2)} kVA`);
}
main();
```

## Alt API'ler

| Modül | Kısa |
| --- | --- |
| [Variable API](/docs/tr/jdk21/platform/scripts/server/variable-api/) | Değişken okuma/yazma, stats |
| [Connection API](/docs/tr/jdk21/platform/scripts/server/connection-api/) | Bağlantı / device / frame |
| [Alarm API](/docs/tr/jdk21/platform/scripts/server/alarm-api/) | Alarm durumları, fired alarm geçmişi |
| [Trend API](/docs/tr/jdk21/platform/scripts/server/trend-api/) | Trend tag yönetimi |
| [Datasource API](/docs/tr/jdk21/platform/scripts/server/datasource-api/) | SQL, InfluxQL |
| [Data Transfer API](/docs/tr/jdk21/platform/scripts/server/datatransfer-api/) | Dosya tabanlı veri aktarımı |
| [Notification API](/docs/tr/jdk21/platform/scripts/server/notification-api/) | Mail, SMS, web notification |
| [Script API](/docs/tr/jdk21/platform/scripts/server/script-api/) | Script zamanlama, global object |
| [Report API](/docs/tr/jdk21/platform/scripts/server/report-api/) | Classic + Jasper rapor |
| [Project API](/docs/tr/jdk21/platform/scripts/server/project-api/) | Proje bilgisi, lokasyon |
| [Log API](/docs/tr/jdk21/platform/scripts/server/log-api/) | Audit log |
| [System API](/docs/tr/jdk21/platform/scripts/server/system-api/) | Shutdown, restart, exec |
| [User API](/docs/tr/jdk21/platform/scripts/server/user-api/) | Kullanıcı listesi |
| [I/O Utils API](/docs/tr/jdk21/platform/scripts/server/io-utils-api/) | REST, ping, file, JSON |
| [Utils API](/docs/tr/jdk21/platform/scripts/server/utils-api/) | uuid, tarih, bit, format |
| [Language API](/docs/tr/jdk21/platform/scripts/server/language-api/) | `loc` |
| [Keyword API](/docs/tr/jdk21/platform/scripts/server/keyword-api/) | Meta veri |
| [Console API](/docs/tr/jdk21/platform/scripts/server/console-api/) | `consoleLog` |
