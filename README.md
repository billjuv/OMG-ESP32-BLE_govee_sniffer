# "Govee Sniffers" — ESP32 Bluetooth BLE to MQTT with OpenMQTTGateway

These are ESP32 D1 Mini boards flashed with OpenMQTTGateway (OMG) `esp32dev-ble` firmware. They listen for all nearby Bluetooth BLE signals and forward the data to your MQTT broker over WiFi.

They fit nicely into a 3D printed case and hang from a USB Micro cable plugged into a small USB charger (5V). No soldering required.

**Primary use case:** Monitoring Govee Thermo-Hygrometer sensors (temperature and humidity) and forwarding readings to Node-RED for dashboards and alerts.

**Real-world deployment:** Used to monitor commercial refrigerators and freezers, with Node-RED push notifications (via Remote-RED) triggered if temperatures fall outside set thresholds.

---

## Hardware

- ESP32 D1 Mini board
- USB Micro cable and 5V USB charger
- [3D printed case — ESP32 D1 Mini Case by briancmoses (Printables.com)](https://www.printables.com/model/342820-esp32-d1-mini-case)

> **Note on the case:** The lid brim hits the reset switch on the board, preventing the lid from fitting properly. Trim a small amount off the brim around the switch to fix this.

<img src=Attachments/5FB23FBC-6F71-4341-87C9-EF0F291E7915.jpeg width="60%"/>

---

## Firmware Installation

Install directly from a Chrome browser — no Arduino IDE needed:

1. Go to [https://docs.openmqttgateway.com/upload/web-install.html](https://docs.openmqttgateway.com/upload/web-install.html)
2. Choose **`ESP32Dev-BLE`** configuration
   - **D1 Mini:** Just connect and flash
   - **DevKit v1:** Hold the BOOT button when plugging in
3. After flashing, reconnect power to open the captive portal
4. Connect to the **`OMG_ESP32_BLE`** WiFi network on your phone
5. Enter your WiFi and MQTT credentials
   - Gateway password: **admin / R&W**

Subsequent configuration changes can be made by navigating to the device's IP address in a browser.

---

## MQTT Configuration via MQTT Explorer

All configuration commands are published via MQTT. Use [MQTT Explorer](https://mqtt-explorer.com/) to send them.

**Command topic** (adjust path to match your device name):
```
home/OMG_ESP32_BLE/commands/MQTTtoBT/config
```

> ⚠️ **Formatting is critical.** Use [jsonformatter.org](https://jsonformatter.org) to validate JSON before publishing. Re-type any curly quotes if copying from a document — they must be straight quotes.

> ⚠️ **Retain checkbox:** Use "Retain" **only** for white lists or black lists. For all other commands, use `"save":true` instead to store settings in ESP flash memory.

---

### White List (limit reporting to specific devices)

Use a white list to restrict MQTT traffic to only the Govee sensors you care about. This is strongly recommended if there are several BLE devices nearby.

Publish with **Retain checked**:

```json
{
   "white-list": [
      "A4:C1:38:2F:89:B0",
      "A4:C1:38:5A:FC:5D",
      "E3:5E:CC:81:C4:80"
   ]
}
```

---

### Speed / Reporting Settings (optional)

The following settings reportedly speed up Govee reporting and limit messages to temperature and iBeacon topics. Publish with **Retain unchecked** (uses `"save":true` to persist in flash):

```json
{
   "adaptivescan": false,
   "interval": 15000,
   "intervalacts": 15000,
   "scanduration": 15000,
   "onlysensors": true,
   "save": true
}
```

> **Note:** Results with these settings may vary — improvement is not always noticeable.

---

## How Govee Devices Report to MQTT

The MQTT topic for each Govee device is based on its MAC address:

```
OMGhome/OMG_ESP32_BLE/BTtoMQTT/A4C1385A13DD
```

<img src=Attachments/2139D691-E4E3-4235-940E-AD0069B67DE9.png width="100%"/>

Example payload from a Govee H5074 Thermo-Hygrometer:

```json
{
   "id": "E3:5E:CC:81:C4:80",
   "name": "Govee_H5074_C480",
   "rssi": -83,
   "brand": "Govee",
   "model": "Thermo-Hygrometer",
   "model_id": "H5074",
   "type": "THB",
   "tempc": 9.63,
   "tempf": 49.334,
   "hum": 31.77,
   "batt": 100
}
```

---

## Node-RED Integration

The included Node-RED flow (`OMG_Govee.json` in the NodeRed folder) splits incoming MQTT messages from the Govee sensors into individual topic messages for temperature and humidity, making them easy to route to a dashboard or database.

**Flow overview:**
- MQTT-in node subscribes to a specific Govee device topic
- Split node separates the JSON payload into individual messages
- Switch node routes `tempf` and `hum` values to separate outputs
- Link Out nodes pass values downstream to dashboard or InfluxDB nodes

<img src=Attachments/77AD7CA5-F47E-4857-9A94-1221D3579624.jpg width="100%"/>

<img src=Attachments/5A2F676D-7AE5-4284-8F90-89177701A9FC.jpg width="60%"/>

<img src=Attachments/D79D3B39-D2FF-4AB8-B345-10A163DFB282.jpg width="60%"/>

> The full Node-RED flow JSON is in the `NodeRed` folder of this repository.

---

## Related Resources

- [OpenMQTTGateway Documentation](https://docs.openmqttgateway.com)
- [OpenMQTTGateway Web Installer](https://docs.openmqttgateway.com/upload/web-install.html)
- [ESP32 D1 Mini Case on Printables](https://www.printables.com/model/342820-esp32-d1-mini-case)
