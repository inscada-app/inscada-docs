---
title: "Secure Access VPN"
description: "WireGuard VPN ile endustriyel tesislere guvenli uzaktan erisim"
sidebar:
  order: 1
---

inSCADA Secure Access, endustriyel tesislerinize her yerden guvenli erisim saglayan bir VPN cozumudur. WireGuard protokolu ile sifreli, hizli ve guvenilir baglanti kurar. Windows masaustu uygulamasi ile tek tikla baglanin — karmasik VPN konfigurasyonlarina gerek yok.

## Ozellikler

- **WireGuard VPN** — Dusuk gecikme, yuksek performans. Geleneksel VPN cozumlerine gore cok daha hizli
- **Tek tikla kurulum** — Token yapistir, baglan. Karmasik konfigurasyona gerek yok
- **Ozel erisim adresi** — Her musteriye ozel subdomain: `sizinfirma.inscada.online`
- **Baglanti izleme** — Durum, uptime ve transfer verilerini gercek zamanli gorun
- **Otomatik yeniden baglanti** — Baglanti koptugunda otomatik olarak yeniden baglanir
- **Windows ile baslat** — Bilgisayar acildiginda VPN baglantiniz hazir

## Baslangic

### 1. Plan Secin

[vpn.inscada.online](https://vpn.inscada.online) adresinden Basic veya Pro planini secin. Odeme sonrasi VPN altyapiniz otomatik olarak hazirlanir.

### 2. Uygulamayi Indirin

[inscada.com/download](https://inscada.com/download/) sayfasindan **inSCADA Secure Access** installer'ini indirip kurun.

**Gereksinimler:** Windows 10/11

### 3. Aktivasyon

Uygulamayi acin ve size gonderilen **aktivasyon token**'ini yapistirarak Activate butonuna basin. VPN konfigurasyonunuz otomatik yuklenir.

![Aktivasyon ekrani — Token veya config dosyasi ile kurulum](../../../../../assets/docs/secure-access-activate.png)

Alternatif olarak bir WireGuard `.conf` dosyasi da icerikten yukleyebilirsiniz (**Import Config File** butonu).

### 4. Baglanin

Connect butonuna basin. Tesisinize `sizinfirma.inscada.online` adresinden guvenle erisin.

![Bagli durum — Site adresi, handshake ve transfer bilgileri](../../../../../assets/docs/secure-access-connected.png)

## Uygulama Arayuzu

Uygulama uc temel durumda calisir:

| Durum | Aciklama |
|-------|----------|
| **Disconnected** | VPN baglantisi kapali. Connect butonu ile baglanin |
| **Connecting** | Baglanti kuruluyor |
| **Connected** | VPN aktif. Site adresi, handshake suresi ve transfer verileri gorunur |

![Baglanti kesik durumu — Connect butonu ile yeniden baglanin](../../../../../assets/docs/secure-access-disconnected.png)

Baglanti kuruldugunda ekranda su bilgiler gorunur:
- **Site** — Tesisinizin erisim adresi (ornegin `test-client.inscada.online`)
- **Status** — Baglanti durumu
- **Handshake** — Son basarili el sikisma suresi
- **Transfer** — Gonderilen ve alinan veri miktari

### Ek Ozellikler

- **Start with Windows** — Bu secenegi isaretleyerek uygulamanin Windows ile birlikte baslamasini saglayabilirsiniz
- **Reset Configuration** — Mevcut VPN yapilandirmasini sifirlar. Yeniden aktivasyon gerektirir
- **Sistem tepsisi** — Uygulama kapatildiginda sistem tepsisinde calismaya devam eder

## Kullanim Senaryolari

- **SCADA bakim & guncelleme** — Tesis sahasina gitmeden SCADA yaziliminizi uzaktan guncelleyin, parametre degisikligi yapin
- **Acil ariza mudahalesi** — Gelen alarm bildiriminde evden tesisinize baglanip sorunu aninda tespit edin
- **Uzak saha izleme** — Cografi olarak daginik tesislerinizi tek noktadan izleyin
- **Coklu tesis yonetimi** — Tum tesislerinizi tek panelden yonetin

## Geleneksel Cozumlerle Karsilastirma

| Kriter | inSCADA Secure Access | Geleneksel VPN |
|--------|----------------------|----------------|
| Kurulum suresi | Dakikalar | Saatler/Gunler |
| Ozel erisim adresi | sizinfirma.inscada.online | IP adresi ile erisim |
| Baglanti performansi | WireGuard (dusuk gecikme) | OpenVPN/IPSec (yuksek gecikme) |
| Otomatik yeniden baglanti | Var | Genellikle yok |
| Yonetim paneli | Web tabanli admin panel | Komut satiri / manuel |

## Guvenlik

- Tum trafik WireGuard ile uctan uca sifrelenir
- VPN konfigurasyonu lokal makinede guvenli sekilde saklanir
- Her site icin ayri anahtar cifti olusturulur
- Baglanti token'lari tek kullanimliktir

## Destek

Sorulariniz icin [inscada.com/contact](https://inscada.com/contact/) sayfasindan bize ulasin veya [vpn.inscada.online](https://vpn.inscada.online) admin panelinden destek talebi olusturun.
