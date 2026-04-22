---
title: "Alarm API"
description: "Alarm ve alarm grubu sorgulama, aktifleştirme, fired alarm geçmişi, acknowledge / comment / force-off işlemleri"
sidebar:
  order: 3
---

Alarm API; alarm ve alarm gruplarının durumunu sorgulamak, aktivasyon/deaktivasyon yapmak, tetiklenmiş alarm (fired alarm) geçmişini çekmek ve alarmlar üzerinde acknowledge / comment / force-off aksiyonları yapmak için kullanılır.

## Alarm ve Grup Meta Verisi

### `ins.getAlarm(name)` / `(projectName, name)`

Tek bir alarmın tanımını döner — `AlarmResponseDto`.

```javascript
var a = ins.getAlarm("HighTemperature");
ins.consoleLog(a.getName() + " (" + a.getType() + ") — delay=" + a.getDelay());
```

### `ins.getAlarmGroup(name)` / `(projectName, name)`

Tek bir alarm grubunun tanımını döner — `AlarmGroupResponseDto`.

```javascript
var g = ins.getAlarmGroup("Temperature_Alarms");
ins.consoleLog(g.getName() + " priority=" + g.getPriority() + " scan=" + g.getScanTimeInMillis() + "ms");
```

### `AlarmResponseDto` Alanları

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` | `String` | Alarm adı |
| `getDsc()` | `String` | Açıklama |
| `getGroupId()` | `String` | Grup ID |
| `getType()` | `String` | Tip (`DIGITAL`, `ANALOG_HIGH_HIGH`, `ANALOG_HIGH`, `ANALOG_LOW`, `ANALOG_LOW_LOW`, `ANALOG_SET_POINT`, `CUSTOM`) |
| `getDelay()` | `Integer` | Tetikleme gecikmesi (saniye) |
| `getIsActive()` | `Boolean` | Alarm tanımı aktif mi |
| `getPart()` | `String` | Grup içi parçalama anahtarı |
| `getOnTimeVariableId()` / `getOffTimeVariableId()` | `String` | On/off zamanının yazıldığı değişken ID'leri |

### `AlarmGroupResponseDto` Alanları

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` | `String` | Grup adı |
| `getDsc()` | `String` | Açıklama |
| `getPriority()` | `Short` | Öncelik |
| `getScanTimeInMillis()` | `Integer` | Grup içi tarama periyodu |
| `getOnScriptId()` / `getOffScriptId()` / `getAckScriptId()` | `String` | On/off/ack anında çalışacak script ID'leri |
| `getOnNoAckColor()` / `getOnAckColor()` / `getOffNoAckColor()` / `getOffAckColor()` | `String` | Durum bazlı renk kodları |
| `getPrinterIp()` / `getPrinterPort()` | — | Yazıcı adresi (varsa) |
| `getPrintWhenOn()` / `getPrintWhenOff()` / `getPrintWhenAck()` / `getPrintWhenComment()` | `Boolean` | Hangi olaylarda yazıcıya düşecek |

## Durum Sorgulama

### `ins.getAlarmStatus(name)` / `(projectName, name)`

`AlarmStatus` enum döner — iki değer:

| Değer | Anlam |
| --- | --- |
| `"Active"` | Alarm aktif olarak çalışıyor |
| `"Not Active"` | Alarm devre dışı |

```javascript
if (ins.getAlarmStatus("HighTemperature") == "Not Active") {
    ins.notify("info", "Alarm", "HighTemperature devre dışı");
}
```

### `ins.getAlarmGroupStatus(name)` / `(projectName, name)`

Grup için aynı enum.

### `ins.getCurrentAlarmGroupInfo(groupName)`

Grubun o anki özet bilgisini döner — `Map<String, Object>` (toplam, aktif, ack'lenmemiş sayıları gibi).

```javascript
var info = ins.getCurrentAlarmGroupInfo("Temperature_Alarms");
ins.consoleLog(JSON.stringify(info));
```

## Grup Aktivasyon / Deaktivasyon

### `ins.activateAlarmGroup(name)` / `(projectName, name)`
### `ins.deactivateAlarmGroup(name)` / `(projectName, name)`

Bakım veya geçici süspansiyon için tüm grubu açıp kapar.

```javascript
// Bakım başlangıcı
ins.deactivateAlarmGroup("Temperature_Alarms");
// ...
// Bakım sonu
ins.activateAlarmGroup("Temperature_Alarms");
```

## Fired Alarm (Tetiklenmiş Alarm) Sorgulama

"Fired alarm" kavramı bir alarmın tetiklenmesinden (on) sönmesine (off) kadar geçen tekil bir olayı temsil eder. İsteğe bağlı olarak sönmüş (off) olanları dahil etmek veya etmemek için `includeOff` parametresi vardır.

### Tekil Fired Alarm

#### `ins.getFiredAlarm()` / `(projectName)` / `(index)` / `(index, includeOff)`

Index'e göre tek bir fired alarm döner.

```javascript
var f = ins.getFiredAlarm(0);                 // en yeni aktif
var fAny = ins.getFiredAlarm(0, true);        // en yeni (off dahil)
```

### Birden Fazla Fired Alarm

#### `ins.getFiredAlarms(index, count)` / `(index, count, includeOff)` / projectName varyantları

Bir pencere döndürür (start = `index`, boyut = `count`) — `Collection<FiredAlarmDto>`.

```javascript
// Son 10 aktif fired alarm
var top10 = ins.getFiredAlarms(0, 10);
top10.forEach(function(f) {
    ins.consoleLog(f.getName() + " @ " + f.getOnTime());
});

// Son 50 (off dahil)
var last50 = ins.getFiredAlarms(0, 50, true);
```

#### `ins.getFiredAlarmsByDate(startDate, endDate, includeOff, limit)` / projectName varyantı

Tarih aralığında tetiklenen alarmlar.

```javascript
var end = ins.now();
var start = ins.getDate(end.getTime() - 86400000);   // son 24 saat
var history = ins.getFiredAlarmsByDate(start, endDate, true, 500);
```

#### `ins.getFiredAlarmsByPart(part, page, count)` / projectName varyantı

Belirli bir `part` (parçalama anahtarı) için sayfalı fired alarm listesi.

```javascript
var forLine1 = ins.getFiredAlarmsByPart("Line1", 0, 20);
```

### Aktif Olanlar (Current Alarms)

#### `ins.getCurrentAlarms(includeOff)` / `(projectName, includeOff)`

O anda aktif olan alarmların tamamı. `includeOff=true` sönmüş olup ack bekleyenleri de kapsar.

```javascript
var active = ins.getCurrentAlarms(false);
ins.setVariableValue("ActiveAlarmCount", { value: active.size() });
```

#### `ins.getCurrentAlarmsByName(alarmNames[], includeOff)`

Verilen alarm adlarından şu an aktif olanların haritası — `Map<String, FiredAlarmDto>`.

```javascript
var map = ins.getCurrentAlarmsByName(["HighTemperature", "LowPressure"], false);
if (map.HighTemperature) {
    ins.consoleLog("HighTemperature aktif, on at " + map.HighTemperature.getOnTime());
}
```

### `FiredAlarmDto` Alanları

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` / `getDsc()` | `String` | Alarm adı ve açıklaması |
| `getId()` / `getAlarmId()` / `getGroupId()` / `getProjectId()` | `String` | Kimlikler |
| `getGroup()` / `getProject()` | `String` | Grup ve proje adı |
| `getPart()` | `String` | Parçalama anahtarı |
| `getStatus()` | `FiredAlarmStatus` | `"On"` / `"Off"` |
| `getStatusValue()` | `Integer` | Ham status tamsayısı |
| `getFiredAlarmType()` | `FiredAlarmType` | `Digital`, `Custom`, `Analog High High`, `Analog High`, `Analog Low`, `Analog Low Low`, `Analog Set Point` |
| `getOnValue()` / `getOffValue()` / `getOnValueB()` / `getOffValueB()` | `Double` | Tetikleme ve sönme değerleri |
| `getOnTime()` / `getOnTimeInMs()` | `Date` / `Long` | Tetikleme zamanı |
| `getOffTime()` / `getOffTimeInMs()` | `Date` / `Long` | Sönme zamanı (henüz sönmediyse `null`) |
| `getAcknowledgeTime()` / `getAcknowledgeTimeInMs()` / `getAcknowledger()` | — | Acknowledge bilgisi |
| `getForcedOff()` / `getForcedOffBy()` | `Boolean` / `String` | Zorla kapatılıp kapatılmadığı |
| `getComment()` / `getCommentedBy()` / `getCommentTime()` / `getCommentTimeInMs()` | — | Yorum bilgisi |

## Alarm Üzerinde Aksiyon

Aksiyonlar (`acknowledgeAlarm`, `forceOffAlarm`, `commentAlarm`) **FiredAlarmDto** bekler — tipik kalıp: önce fired alarm'ı bul, sonra üzerinde işlem yap.

### `ins.acknowledgeAlarm(firedAlarmDto)`

```javascript
var active = ins.getCurrentAlarms(false);
active.forEach(function(f) {
    ins.acknowledgeAlarm(f);
});
```

### `ins.forceOffAlarm(firedAlarmDto)`

Alarmı zorla söndürür — gerçek tetikleyici koşul sürüyor olsa bile.

```javascript
var f = ins.getFiredAlarm(0);
if (f && f.getName() == "StaleSensor") {
    ins.forceOffAlarm(f);
}
```

### `ins.commentAlarm(firedAlarmDto, comment)`

Alarma operatör yorumu ekler.

```javascript
var f = ins.getFiredAlarm(0);
ins.commentAlarm(f, "Sensör kalibrasyonu yapıldı, false positive");
```

## Alarm Tanımını Güncelleme

### `ins.updateAlarm(alarmName, dto)`

Tam `AlarmResponseDto` bekler — get/mutate/update kalıbı kullan.

```javascript
var a = ins.getAlarm("HighTemperature");
a.setDelay(60);                 // gecikme 60 sn
a.setDsc("Yüksek sıcaklık — güncellendi");
ins.updateAlarm("HighTemperature", a);
```

## Örnek: Alarm Raporu ve Oto-Ack

```javascript
function main() {
    var end = ins.now();
    var start = ins.getDate(end.getTime() - 3600000);

    // Son 1 saatte tetiklenen tüm alarmlar
    var events = ins.getFiredAlarmsByDate(start, end, true, 1000);
    ins.writeLog("INFO", "AlarmReport", "Son 1 saat: " + events.size() + " olay");

    // Sensör temizliği sonrası aktif olanları otomatik ack'le
    var active = ins.getCurrentAlarms(false);
    active.forEach(function(f) {
        if (f.getGroup() == "Sensors_Cleanup") {
            ins.commentAlarm(f, "Oto-ack — temizlik sonrası");
            ins.acknowledgeAlarm(f);
        }
    });
}
main();
```
