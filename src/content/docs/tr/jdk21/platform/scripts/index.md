---
title: "Script Engine"
description: "inSCADA JDK21 script motoru — server-side ins.* ve client-side Inscada.* API'lerine genel bakış"
sidebar:
  order: 0
  label: "Genel Bakış"
---

inSCADA JDK21, iki ayrı script ortamı sunar: **server-side** (`ins.*`) ve **client-side** (`Inscada.*`). İkisi farklı yerlerde çalışır, farklı amaçlar için tasarlanmıştır ve birbirinden izole API yüzeyleri vardır.

## İki Ayrı Dünya

|  | Server-side (`ins.*`) | Client-side (`Inscada.*`) |
| --- | --- | --- |
| **Nerede çalışır** | inSCADA sunucusu (JVM) | Tarayıcı (browser JS) |
| **Engine** | GraalJS (Nashorn uyumlu) | Tarayıcının kendi motoru |
| **Binding** | `ins` (global) | `new InscadaApi(projectName)` |
| **Amaç** | Otomasyon, hesap, alarm, zamanlı iş | Kullanıcı etkileşimi, UI kontrolü |
| **Tipik kullanım** | Scheduled script, variable / alarm / log expression, MQTT parse | Animation click, custom HTML widget, dashboard widget |

→ [Server-side (ins.*) detayları](/docs/tr/jdk21/platform/scripts/server/)
→ [Client-side (Inscada.*) detayları](/docs/tr/jdk21/platform/scripts/client/)

## Nerede Kullanılır?

inSCADA'da script'ler pek çok yerde devreye girer. Aşağıdaki tablo her kullanım yerinin hangi ortamda çalıştığını gösterir.

| Kullanım Yeri | Ortam | Tetikleme |
| --- | --- | --- |
| **Scheduled Script** | server (`ins.*`) | Periodic / Daily / Once / manuel |
| **Variable Expression** | server (`ins.*`) | Poll döngüsünde otomatik |
| **Alarm Expression** | server (`ins.*`) | Değer değişiminde otomatik |
| **Log Expression** | server (`ins.*`) | Poll döngüsünde otomatik |
| **MQTT Subscribe/Publish** | server (`ins.*`) | Her MQTT mesajında |
| **Animation Expression** | server (`ins.*`) | UI güncellemesinde (uzaktan) |
| **Animation Click Script** | client (`Inscada.*`) | Kullanıcı tıklaması |
| **Custom HTML Widget** | client (`Inscada.*`, sandboxed iframe) | Widget kendi içinde |
| **Dashboard Widget** | client (`Inscada.*`) | Dashboard render / etkileşim |

:::tip[Hangisini seçmeliyim?]
Veri yazıp/okuyor, alarm tetikliyor, zamanlanmış bir hesap yapıyorsan **server-side** (`ins.*`). Kullanıcıya popup gösterecek, SVG zoom yapacak, animasyon değiştireceksen **client-side** (`Inscada.*`). İkisini birleştirmek istersen: client'ın mirror metodları server metodlarını async çağırır.
:::

## Sandbox ve Güvenlik

İki ortamda da script'ler izole çalışır, ama kısıtlar farklıdır:

- **Server tarafı** — `HostAccess.EXPLICIT`: yalnızca `@HostAccess.Export` işaretli Java metodlarına erişim. `eval`, `with` kapalı. Thread/IO/process/native yasak. 100k statement limiti, 60s timeout (default).
- **Client tarafı** — Custom HTML widget'lar sandboxed iframe'de çalışır; ana sayfa API'sine `InscadaBridge` üzerinden, origin-doğrulamalı `postMessage` ile erişir.

:::caution
Script'ler platform içinde çalışır. Sonsuz döngü veya çok ağır hesaplama performansı etkiler. Server-side'da 100k statement / 60s default limit devrededir; client-side'da tarayıcı sekmesini donduracak kod yazmaktan kaçının.
:::
