---
title: "Configuration"
description: "application.yml parametreleri, dizin yapısı, NSSM servis kurulumu ve SSL sertifika yönetimi"
sidebar:
  order: 0
---

inSCADA tek bir **JAR dosyası** olarak dağıtılır. Tüm yapılandırma parametreleri JAR içindeki `application.yml` dosyasında varsayılan değerleriyle gömülüdür.

## Kurulum Dizin Yapısı

Windows kurulumu sonrası dizin yapısı:

```
C:\Program Files\inSCADA\
├── inscada.jar                          ← platform (tek dosya)
├── inscada-keystore.p12                 ← SSL sertifika
├── inscada.cer                          ← dışa aktarılmış public sertifika
├── application.yml                      ← özelleştirilmiş yapılandırma (opsiyonel)
├── inSCADA_Service_Install.bat          ← NSSM servis kurulum scripti
├── inSCADA_TimeserisDB_Service_Install.bat ← InfluxDB servis kurulum scripti
├── firewallsettings.bat                 ← Windows Firewall kural scripti
├── create-cert.bat                      ← SSL sertifika oluşturma (interaktif)
├── auto-create-cert.bat                 ← SSL sertifika oluşturma (otomatik)
├── files\                               ← dosya depolama (SVG, rapor, ekler)
├── logs\                                ← uygulama logları
│   └── log.log
└── influxdb-1.8.3-1\                   ← InfluxDB zaman serisi veritabanı
    ├── influxd.exe
    ├── influx.exe
    ├── influxdb.conf
    └── TimeseriesData\                  ← tarihsel veri dosyaları

C:\Program Files\OpenJDK\
└── jdk-11.0.18.10-hotspot\              ← Java 11 runtime
    └── bin\java.exe

C:\Program Files\PostgreSQL\12\          ← PostgreSQL veritabanı
    └── bin\
```

## Yapılandırmayı Özelleştirme

Varsayılan ayarları değiştirmenin iki yolu vardır:

### 1. Harici application.yml (opsiyonel)

JAR'ın yanına (aynı dizine) bir `application.yml` dosyası koyarsanız, Spring Boot harici dosyayı otomatik algılar ve JAR içindeki varsayılanların üzerine yazar. Yalnızca değiştirmek istediğiniz parametreleri yazmanız yeterlidir:

```yaml
# C:\Program Files\inSCADA\application.yml
spring:
  datasource:
    url: jdbc:postgresql://192.168.1.100:5432/promis
    password: guclu_sifre
server:
  http:
    redirect: true
```

### 2. JVM -D parametreleri (NSSM ile)

Kurulum scripti bu yöntemi kullanır — parametreler doğrudan NSSM üzerinden `-D` flag'leri ile geçirilir:

```
-Dserver.ssl.key-store="C:\Program Files\inSCADA\inscada-keystore.p12"
-Dserver.ssl.key-store-password=19051905
-Dspring.influxdb.username=inscada
-Dspring.influxdb.password=19051905
-Dspring.profiles.active=dev
-Duser.timezone=Europe/Istanbul
-Dins.files.path="C:\Program Files\inSCADA\files"
```

:::tip
`-D` parametreleri `application.yml` dosyasındaki değerlerin üzerine yazar. Her iki yöntem birlikte de kullanılabilir.
Yapılandırma değişikliklerinin uygulanması için servis yeniden başlatılmalıdır.
:::

---

## application.yml Parametreleri

### Veritabanı (PostgreSQL)

```yaml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/promis
    username: postgres
    password: 1907
    minimum-idle: 10
    maximum-pool-size: 25
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **url** | `jdbc:postgresql://localhost:5432/promis` | PostgreSQL bağlantı URL'si. Uzak sunucu için `localhost` yerine IP yazın |
| **username** | `postgres` | Veritabanı kullanıcı adı |
| **password** | `1907` | Veritabanı şifresi |
| **minimum-idle** | `10` | Havuzdaki minimum boş bağlantı sayısı |
| **maximum-pool-size** | `25` | Maksimum eşzamanlı bağlantı sayısı |

```yaml
  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        show_sql: false
        default_schema: inscada
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
    hibernate:
      ddl-auto: none
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **default_schema** | `inscada` | PostgreSQL şema adı |
| **show_sql** | `false` | SQL sorgularını loga yaz (debug için) |
| **batch_size** | `20` | Toplu INSERT/UPDATE batch boyutu |
| **ddl-auto** | `none` | Şema otomatik oluşturma kapalı (Flyway yönetir) |

### Zaman Serisi Veritabanı (InfluxDB)

```yaml
spring:
  influxdb:
    database: inscada
    url: http://localhost:8086
    username: inscada
    password: 19051905
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **url** | `http://localhost:8086` | InfluxDB HTTP API adresi |
| **database** | `inscada` | InfluxDB veritabanı adı |
| **username** | `inscada` | InfluxDB kullanıcı adı |
| **password** | — | InfluxDB şifresi |

InfluxDB 1.8 kullanılır ve kurulum dizini altında (`influxdb-1.8.3-1/`) yer alır. Değişken logları (tarihsel veriler) burada tutulur.

#### InfluxDB Retention Policy'leri

Kurulum sırasında otomatik oluşturulan saklama politikaları:

| Retention Policy | Süre | Veri Tipi |
|-----------------|------|-----------|
| **autogen** | Sınırsız | Varsayılan |
| **variable_value_rp** | 365 gün | Değişken değerleri |
| **event_log_rp** | 14 gün | Olay logları |
| **fired_alarm_rp** | 365 gün | Alarm geçmişi |
| **auth_attempt_rp** | 365 gün | Giriş denemeleri |

### Cache (Redis)

```yaml
spring:
  redis:
    host: localhost
    port: 6379
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **host** | `localhost` | Redis sunucu adresi |
| **port** | `6379` | Redis port numarası |

Redis, anlık değişken değerlerinin cache'lenmesi ve gerçek zamanlı veri dağıtımı için kullanılır.

### Sunucu & Port Ayarları

```yaml
server:
  port: 8082
  ssl:
    enabled: true
    key-alias: inscada
    key-store: "C:\\Program Files\\inSCADA\\inscada-keystore.p12"
    key-store-type: PKCS12
    key-store-password: 19051905
  http:
    redirect: false
    port: 8081
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **server.port** | `8082` | HTTPS port numarası |
| **server.http.port** | `8081` | HTTP port numarası |
| **server.http.redirect** | `false` | HTTP → HTTPS yönlendirme |
| **ssl.enabled** | `true` | SSL/TLS etkin |
| **ssl.key-alias** | `inscada` | Sertifika alias'ı |
| **ssl.key-store** | — | SSL sertifika dosyası (PKCS12) yolu |
| **ssl.key-store-password** | — | Keystore şifresi |

#### Port Tablosu

| Port | Protokol | Kullanım |
|------|----------|----------|
| **8081** | HTTP | Web arayüzü ve REST API |
| **8082** | HTTPS | Web arayüzü ve REST API (şifreli) |
| **8086** | HTTP | InfluxDB API (dahili) |
| **5432** | TCP | PostgreSQL (dahili) |
| **6379** | TCP | Redis (dahili) |

### Platform Ayarları (ins.*)

```yaml
ins:
  node:
    id: ins01
  files:
    path: "C:\\Program Files\\inSCADA\\files"
  jwt:
    secret:
  accessToken:
    duration: 5
  refreshToken:
    duration: 1
  job:
    executor:
      max-threads: 1000
  variable:
    value:
      list:
        size: 300
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **ins.node.id** | `ins01` | Node tanımlayıcı (cluster ortamında benzersiz olmalı) |
| **ins.files.path** | `./fs` | Dosya depolama dizini (SVG, rapor, ek dosyalar) |
| **ins.jwt.secret** | _(otomatik)_ | JWT token imzalama anahtarı. Boş bırakılırsa rastgele üretilir |
| **ins.accessToken.duration** | `5` | Access token süresi (dakika) |
| **ins.refreshToken.duration** | `1` | Refresh token süresi (gün) |
| **ins.job.executor.max-threads** | `1000` | Maksimum eşzamanlı iş parçacığı (script, bağlantı, rapor vb.) |
| **ins.variable.value.list.size** | `300` | Değişken son değer listesi boyutu |

### Mesaj Kuyruğu (Apache Artemis)

```yaml
spring:
  artemis:
    enabled: false
    mode: NATIVE
  remote:
    broker:
      enabled: false
      list:
        - host: 192.168.1.5
          port: 61616
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **artemis.enabled** | `false` | Dahili Artemis broker. Tek makine kurulumunda `false` |
| **artemis.mode** | `NATIVE` | Broker modu |
| **remote.broker.enabled** | `false` | Uzak broker kullanımı (cluster/HA yapılandırması) |
| **remote.broker.list** | — | Uzak broker listesi (host + port) |

:::note
Artemis, yalnızca yedekli mimari (HA) veya cluster yapılandırmasında etkinleştirilir. Tek makine kurulumunda dahili in-memory mesajlaşma kullanılır.
:::

### Cluster Yapılandırması

```yaml
ins:
  cluster:
    enabled: false
    redis:
      replication:
        enabled: false
```

Cluster yapılandırması hakkında detaylı bilgi için [Yedekli Mimari](/docs/tr/deployment/redundancy/) sayfasına bakın.

### Dosya Yükleme Limiti

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 256MB
      max-request-size: 256MB
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| **max-file-size** | `256MB` | Tekil dosya maksimum boyutu |
| **max-request-size** | `256MB` | İstek başına maksimum boyut |

### Zaman Dilimi

```
-Duser.timezone=Europe/Istanbul
```

Servis kurulum scriptinde JVM parametresi olarak belirtilir. Tarih/saat gösterimleri ve zamanlama işlemleri bu zaman dilimine göre çalışır.

### Veritabanı Migrasyon (Flyway)

```yaml
spring:
  flyway:
    schemas: inscada
    table: schema_version
```

Flyway, veritabanı şemasını otomatik yönetir. Güncelleme sırasında yeni migration'lar otomatik uygulanır. Manuel müdahale gerekmez.

---

## Profiller

inSCADA üç yapılandırma profili destekler:

### Varsayılan (default)

Özel profil belirtilmediğinde kullanılır.

### dev (Geliştirme)

```yaml
spring:
  profiles: dev
  jpa:
    properties:
      hibernate:
        show_sql: false
        use_sql_comments: true
        format_sql: true
```

SQL logları etkinleşir. Geliştirme ve hata ayıklama için kullanılır. **Kurulum scriptinde varsayılan olarak `dev` profili ayarlıdır.**

### test (Test)

```yaml
spring:
  profiles: test
  datasource:
    url: jdbc:postgresql://192.168.2.123:5432/promis
  influxdb:
    url: http://192.168.2.100:9096
    relay: true
    cluster: influxdb_cluster
    gzip: true
  redis:
    sentinel:
      master: insredis
      nodes: 192.168.2.154:26379, 192.168.2.118:26379, 192.168.2.136:26379
```

InfluxDB Relay, Redis Sentinel ve uzak veritabanı ile yüksek erişilebilirlik test ortamı.

### production

```yaml
spring:
  profiles: production
  influxdb:
    gzip: true
```

InfluxDB GZIP sıkıştırma etkinleşir. Ağ trafiğini azaltır.

### Profil Değiştirme

```bat
:: NSSM ile profil değiştirme
nssm set inSCADA AppParameters "... -Dspring.profiles.active=production ..."

:: Veya komut satırından
java -jar inscada.jar --spring.profiles.active=production
```

---

## NSSM ile Windows Servis Kurulumu

**NSSM** (Non-Sucking Service Manager), inSCADA'yı Windows servisi olarak çalıştırır. Platform, bilgisayar açıldığında otomatik başlar ve arka planda çalışır.

### Hazır Kurulum Scripti

Kurulum dizininde `inSCADA_Service_Install.bat` dosyası bulunur. Yönetici olarak çalıştırmanız yeterlidir:

```bat
:: Yönetici olarak çalıştırın
"C:\Program Files\inSCADA\inSCADA_Service_Install.bat"
```

Bu script şunları yapar:
1. Mevcut inSCADA servisini durdurur ve kaldırır (varsa)
2. Yeni servisi NSSM ile oluşturur
3. Tüm JVM ve yapılandırma parametrelerini ayarlar
4. Otomatik başlatma ve hata durumunda yeniden başlatma yapılandırır
5. Servisi başlatır

### Servis Scripti İçeriği

```bat
@echo off
setlocal

set SERVICE_NAME="inSCADA"
set DISPLAY_NAME="inSCADA Service"
set JAVA_EXE="C:\Program Files\OpenJDK\jdk-11.0.18.10-hotspot\bin\java.exe"
set APP_DIR="C:\Program Files\inSCADA"

set APP_PARAMS="-Dserver.ssl.key-alias=inscada ^
  -Dserver.ssl.key-store-password=19051905 ^
  -Dserver.ssl.key-store=\"C:\Program Files\inSCADA\inscada-keystore.p12\" ^
  -Dspring.influxdb.username=inscada ^
  -Dspring.influxdb.password=19051905 ^
  -Dspring.servlet.multipart.max-request-size=256MB ^
  -Dspring.servlet.multipart.max-file-size=256MB ^
  -Dins.variable.value.list.size=300 ^
  -Djava.library.path=C:\Windows\system32 ^
  -Dspring.profiles.active=dev ^
  -Duser.timezone=Europe/Istanbul ^
  -Dspring.datasource.url=jdbc:postgresql://localhost:5432/promis ^
  -Dins.files.path=\"C:\Program Files\inSCADA\files\" ^
  -jar \"C:\Program Files\inSCADA\inscada.jar\""

nssm stop %SERVICE_NAME% >nul 2>&1
nssm remove %SERVICE_NAME% confirm >nul 2>&1

nssm install %SERVICE_NAME% %JAVA_EXE%
nssm set %SERVICE_NAME% DisplayName "%DISPLAY_NAME%"
nssm set %SERVICE_NAME% AppDirectory %APP_DIR%
nssm set %SERVICE_NAME% AppParameters %APP_PARAMS%
nssm set %SERVICE_NAME% ObjectName LocalSystem
nssm set %SERVICE_NAME% Start SERVICE_AUTO_START
nssm set %SERVICE_NAME% AppExit Default Restart
nssm set %SERVICE_NAME% AppRestartDelay 5000

nssm start %SERVICE_NAME%
```

:::note
Script, hata durumunda servisi 5 saniye bekleyerek otomatik yeniden başlatır (`AppExit Default Restart`, `AppRestartDelay 5000`).
:::

### InfluxDB Servis Kurulumu

InfluxDB de ayrı bir Windows servisi olarak çalışır:

```bat
:: Yönetici olarak çalıştırın
"C:\Program Files\inSCADA\inSCADA_TimeserisDB_Service_Install.bat"
```

Bu script `inSCADA TimeSeries DB` adında bir servis oluşturur:

| Ayar | Değer |
|------|-------|
| **Executable** | `C:\Program Files\inSCADA\influxdb-1.8.3-1\influxd.exe` |
| **Config** | `influxdb-1.8.3-1\influxdb.conf` |
| **Start** | Otomatik |
| **Restart** | Hata durumunda otomatik (5s gecikme) |

### Manuel Servis Oluşturma

Özel parametrelerle servis kurmak isterseniz:

```bat
:: NSSM'i indirin: https://nssm.cc/download
:: nssm.exe'yi PATH'e ekleyin veya tam yol ile çalıştırın

:: Servisi oluştur
nssm install inSCADA "C:\Program Files\OpenJDK\jdk-11.0.18.10-hotspot\bin\java.exe"

:: Çalışma dizini
nssm set inSCADA AppDirectory "C:\Program Files\inSCADA"

:: JVM parametreleri — ihtiyaca göre düzenleyin
nssm set inSCADA AppParameters ^
  "-Xms512m -Xmx4096m ^
  -Dserver.ssl.key-alias=inscada ^
  -Dserver.ssl.key-store=\"C:\Program Files\inSCADA\inscada-keystore.p12\" ^
  -Dserver.ssl.key-store-password=19051905 ^
  -Dspring.profiles.active=production ^
  -Duser.timezone=Europe/Istanbul ^
  -jar \"C:\Program Files\inSCADA\inscada.jar\""

:: Otomatik başlatma & hata yönetimi
nssm set inSCADA DisplayName "inSCADA Service"
nssm set inSCADA Start SERVICE_AUTO_START
nssm set inSCADA AppExit Default Restart
nssm set inSCADA AppRestartDelay 5000

:: Başlat
nssm start inSCADA
```

### JVM Bellek Parametreleri

| Parametre | Açıklama | Önerilen |
|-----------|----------|----------|
| **-Xms** | Başlangıç heap belleği | `512m` |
| **-Xmx** | Maksimum heap belleği | `2048m` (2 GB) |

Büyük projeler (1000+ değişken, çok sayıda bağlantı) için:

```
-Xms1024m -Xmx4096m
```

### Servis Yönetimi

```bat
:: Servisi başlat / durdur / yeniden başlat
nssm start inSCADA
nssm stop inSCADA
nssm restart inSCADA

:: Durum kontrol
nssm status inSCADA

:: Yapılandırmayı düzenle (GUI)
nssm edit inSCADA

:: Servisi kaldır
nssm remove inSCADA confirm
```

Windows standart komutları ile:

```bat
net start inSCADA
net stop inSCADA
sc query inSCADA

:: veya services.msc ile grafiksel yönetim
```

---

## SSL Sertifika Yönetimi

### Otomatik Sertifika Oluşturma

Kurulum dizinindeki `auto-create-cert.bat` scripti:
- Self-signed PKCS12 keystore oluşturur
- SAN (Subject Alternative Name) ile localhost, bilgisayar adı ve IP ekler
- Public sertifikayı `inscada.cer` olarak dışa aktarır
- Sertifikayı Windows Trusted Root'a otomatik ekler

```bat
:: Yönetici olarak çalıştırın
"C:\Program Files\inSCADA\auto-create-cert.bat"
```

### İnteraktif Sertifika Oluşturma

`create-cert.bat` scripti özel değerler girmenizi sağlar:

```bat
"C:\Program Files\inSCADA\create-cert.bat"
:: Keystore dosya adı, şifre, CN, IP ve hostname sorar
```

### Özel Sertifika Kullanımı

Kendi CA sertifikanızı kullanmak için:

```bash
# PEM'den PKCS12 oluşturun
keytool -importkeystore -srckeystore my-cert.pfx \
  -srcstoretype PKCS12 -destkeystore inscada-keystore.p12 \
  -deststoretype PKCS12 -alias inscada

# Veya OpenSSL ile
openssl pkcs12 -export \
  -in certificate.crt -inkey private.key \
  -out inscada-keystore.p12 -name inscada
```

Sonra NSSM parametrelerini güncelleyin:

```bat
nssm set inSCADA AppParameters "... -Dserver.ssl.key-store=\"C:\Program Files\inSCADA\inscada-keystore.p12\" -Dserver.ssl.key-store-password=yeni_sifre ..."
nssm restart inSCADA
```

---

## Windows Firewall Kuralları

Kurulum dizinindeki `firewallsettings.bat` scripti gerekli port kurallarını ekler:

```bat
:: Yönetici olarak çalıştırın
"C:\Program Files\inSCADA\firewallsettings.bat"
```

Eklenen kurallar:

| Kural | Yön | Port | Protokol |
|-------|-----|------|----------|
| inSCADA HTTP (8081) | Gelen + Giden | 8081 | TCP |
| inSCADA HTTPS (8082) | Gelen + Giden | 8082 | TCP |

---

## Yapılandırma Kontrol Listesi

Yeni kurulum sonrası kontrol edilmesi gereken parametreler:

| Parametre | Kontrol |
|-----------|---------|
| `spring.datasource.password` | Varsayılan şifre değiştirildi mi? |
| `server.ssl.key-store-password` | Sertifika şifresi güçlü mü? |
| `spring.influxdb.password` | InfluxDB şifresi güvenli mi? |
| `ins.jwt.secret` | Üretim ortamında güçlü bir secret tanımlandı mı? |
| `ins.accessToken.duration` | Güvenlik gereksinimlerine uygun mu? (varsayılan: 5 dk) |
| `server.http.redirect` | Üretimde `true` olarak ayarlandı mı? |
| `user.timezone` | Doğru zaman dilimi ayarlı mı? |
| JVM `-Xmx` | Proje boyutuna uygun bellek ayrıldı mı? |
| Firewall | `firewallsettings.bat` çalıştırıldı mı? |
| SSL Sertifika | Sertifika Trusted Root'a eklendi mi? |
