---
title: "Connection API"
description: "Haberleşme bağlantılarını başlatma, durdurma, sorgulama ve yapılandırma güncelleme"
sidebar:
  order: 2
---

Connection API, script'ler içinden haberleşme bağlantılarını (Modbus, OPC-UA, IEC-104, S7, MQTT, BACnet vs.) yönetir ve **Connection → Device → Frame** hiyerarşisini okuyup günceller.

## Bağlantı Kontrolü

### `ins.startConnection(name)` / `ins.startConnection(projectName, name)`

Bağlantıyı başlatır. `projectName` verilmezse mevcut proje varsayılır.

```javascript
ins.startConnection("MODBUS-PLC");
ins.startConnection("otherProject", "MODBUS-PLC");
```

### `ins.stopConnection(name)` / `ins.stopConnection(projectName, name)`

Bağlantıyı durdurur.

```javascript
ins.stopConnection("MODBUS-PLC");
```

### `ins.getConnectionStatus(name)` / `ins.getConnectionStatus(projectName, name)`

`ConnectionStatus` enum döner — yalnızca iki değer vardır:

| Değer | Anlam |
| --- | --- |
| `"Connected"` | Bağlantı aktif |
| `"Disconnected"` | Bağlantı kapalı veya koptu |

```javascript
var status = ins.getConnectionStatus("MODBUS-PLC");
if (status == "Disconnected") {
    ins.notify("warning", "Bağlantı", "MODBUS-PLC bağlı değil");
}
```

## Bağlantı, Device, Frame Bilgisi Alma

### `ins.getConnection(name)` / `ins.getConnection(projectName, name)`

`ConnectionResponseDto` döner.

Alanlar:

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` | `String` | Bağlantı adı |
| `getDsc()` | `String` | Açıklama |
| `getProjectId()` | `String` | Proje ID |
| `getProtocol()` | `Protocol` | Protokol (MODBUS, OPC_UA, S7, vs.) |
| `getIp()` | `String` | IP adresi |
| `getPort()` | `Integer` | Port |
| `getConfig()` | `Map<String, Object>` | Protokole özel ek ayarlar |

```javascript
var c = ins.getConnection("MODBUS-PLC");
ins.consoleLog(c.getProtocol() + " " + c.getIp() + ":" + c.getPort());
```

### `ins.getDevice(connectionName, deviceName)`

`DeviceResponseDto` döner.

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` | `String` | Device adı |
| `getDsc()` | `String` | Açıklama |
| `getConnectionId()` | `String` | Üst bağlantı ID |
| `getProtocol()` | `Protocol` | Protokol |
| `getConfig()` | `Map<String, Object>` | Device'e özel ayarlar (slave ID, node ID, vs.) |

```javascript
var d = ins.getDevice("MODBUS-PLC", "Device1");
ins.consoleLog("Slave: " + d.getConfig().slaveId);
```

### `ins.getFrame(connectionName, deviceName, frameName)`

`FrameResponseDto` döner.

| Metod | Tür | Açıklama |
| --- | --- | --- |
| `getName()` | `String` | Frame adı |
| `getDsc()` | `String` | Açıklama |
| `getDeviceId()` | `String` | Üst device ID |
| `getProtocol()` | `Protocol` | Protokol |
| `getMinutesOffset()` | `Integer` | Dakika offset (scan pencereleri için) |
| `getScanTimeFactor()` | `Integer` | Tarama süresi çarpanı |
| `getIsReadable()` | `Boolean` | Okuma aktif mi |
| `getIsWritable()` | `Boolean` | Yazma aktif mi |
| `getConfig()` | `Map<String, Object>` | Frame-specific (örn. Modbus adres aralığı) |

```javascript
var f = ins.getFrame("MODBUS-PLC", "Device1", "HoldingRegs_0_100");
ins.consoleLog("Scan x" + f.getScanTimeFactor() + " — offset " + f.getMinutesOffset() + "m");
```

## Yapılandırma Güncelleme

Güncelleme metodları **tam DTO** ister — tipik kalıp: önce `getConnection` / `getDevice` / `getFrame` ile oku, ilgili alanı değiştir, sonra `updateX` ile geri yaz.

### `ins.updateConnection(connectionName, dto)`

```javascript
var c = ins.getConnection("MODBUS-PLC");
c.setIp("192.168.1.100");
c.setPort(502);
ins.updateConnection("MODBUS-PLC", c);
```

### `ins.updateDevice(connectionName, deviceName, dto)`

```javascript
var d = ins.getDevice("MODBUS-PLC", "Device1");
d.setDsc("Main switchgear — updated");
ins.updateDevice("MODBUS-PLC", "Device1", d);
```

### `ins.updateFrame(connectionName, deviceName, frameName, dto)`

```javascript
var f = ins.getFrame("MODBUS-PLC", "Device1", "HoldingRegs_0_100");
f.setScanTimeFactor(5);           // tarama aralığını seyrekleştir
f.setIsWritable(false);            // salt okunur yap
ins.updateFrame("MODBUS-PLC", "Device1", "HoldingRegs_0_100", f);
```

:::caution
Güncelleme çağrıları çalışan bağlantıyı etkileyebilir — yoğun saha ortamında önce `stopConnection`, güncelle, sonra `startConnection` sırası güvenlidir.
:::

## Örnek: Bağlantıyı IP değişikliği ile yenile

```javascript
function main() {
    var c = ins.getConnection("MODBUS-PLC");
    if (c.getIp() == "192.168.1.100") {
        ins.consoleLog("IP zaten güncel");
        return;
    }

    ins.stopConnection("MODBUS-PLC");
    c.setIp("192.168.1.100");
    ins.updateConnection("MODBUS-PLC", c);
    ins.startConnection("MODBUS-PLC");
    ins.writeLog("INFO", "Connection", "MODBUS-PLC → 192.168.1.100");
}
main();
```
