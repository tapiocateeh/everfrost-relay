# EverFrost relay

ESPHome config that turns a classic ESP32 dev board into a Wi-Fi relay for an
**Anker EverFrost Powered Cooler 40** (model A17A1), which only speaks Bluetooth LE.

It connects to the cooler, asks for its status every 30 s and publishes:

- Cooler battery (%)
- Cooler temperature and target temperature (°F)
- Cooler connected, Cooler signal (RSSI), MQTT connected
- Cooler raw status (hex of the last status packet, for decoding more fields)

Readings go to Home Assistant (native API), MQTT, and a built-in web page.
It is read-only: the only thing it ever sends the cooler is the status request.

## Protocol

| | |
|---|---|
| Service | `0x0157F5DA` |
| Write (no response) | `0x7777` |
| Notify | `0x8888` |
| Status request | `08 EE 00 00 00 01 01 0A 00 02` |
| Reply | 54 bytes, `09 FF` header, byte 7 = length, last byte = sum of the others mod 256 |
| Fields | byte 13 battery %, byte 17 current temp, byte 18 target temp (°F + 128) |

Builds on the EverFrost 30/50 reverse-engineering in
[whoamib2/esphome-anker-everfrost](https://github.com/whoamib2/esphome-anker-everfrost).

## Using it

1. Change `mac_address` under `ble_client:` to your cooler's Bluetooth address.
2. Provide these in your ESPHome `secrets.yaml`: `wifi_ssid`, `wifi_password`, `api_encryption_key`,
   `mqtt_broker`, `mqtt_username`, `mqtt_password` (or remove the `mqtt:` block).
3. Flash once over USB; later updates go over Wi-Fi.
