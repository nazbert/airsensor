# airsensor

Firmware for an indoor air-quality sensor that you build with an ESP32-S3 board and a Sensirion SEN66.

This repository holds the firmware images only. Download an image from [Releases](https://github.com/nazbert/airsensor/releases), flash it on your own hardware, and the device keeps itself up to date from the same place.

## What the firmware does

- Reads the SEN66: PM1.0, PM2.5, PM4.0, PM10, temperature, humidity, VOC index, NOx index and CO2.
- Shows the values, an air-quality index and a history graph on the display.
- Publishes the values as JSON to your own MQTT broker.
- Announces itself to Home Assistant through MQTT discovery. Sample interval, auto-rotate, brightness and auto-update are settable from Home Assistant.
- Takes Wi-Fi and broker settings from a setup page on the device. Nothing about your network is in the image.
- Updates itself from the releases of this repository. You can turn that off.

## Supported hardware

Each release has one image for each board.

| Image | Board | Display | Flash |
|---|---|---|---|
| `esp32-s3-battery` | ESP32-S3 board with an onboard PCF85063A real-time clock (NULA DeepSleep ESP32-S3) | SSD1306 OLED, 0.96 inch, 128 x 64, I2C | 8 MB |
| `esp32-s3-round` | Waveshare ESP32-S3-Touch-LCD-1.28 | Onboard round GC9A01 with touch | 16 MB |

Parts for both: one Sensirion SEN66 (I2C address `0x6B`). The `esp32-s3-battery` build also needs the OLED and one or two push buttons. All parts work at 3.3 V.

### Wiring

| Signal | `esp32-s3-battery` | `esp32-s3-round` |
|---|---|---|
| SEN66 I2C | GPIO 8 and GPIO 9, shared with the OLED | GPIO 17 and GPIO 18 |
| OLED I2C | GPIO 8 and GPIO 9, address `0x3C` or `0x3D` | Not applicable |
| Button 1 | GPIO 7 to ground | The BOOT button (GPIO 0) |
| Button 2 (optional) | GPIO 6 to ground | Not applicable |

On the round board, do not connect the SEN66 to GPIO 6 and GPIO 7. The onboard motion sensor uses the same I2C address on that bus.

## Flash the firmware

You need a USB cable and [esptool](https://docs.espressif.com/projects/esptool/).

1. Download `airsensor-<image>-<version>-factory.bin` for your board from the [latest release](https://github.com/nazbert/airsensor/releases/latest).
2. Connect the board with USB.
3. Write the image at address 0:

```bash
esptool.py --chip esp32s3 --port /dev/ttyACM0 write_flash 0x0 airsensor-esp32-s3-battery-v1.2.3-factory.bin
```

Replace the port and the file name with yours. On Windows the port is a name such as `COM5`.

The file without `-factory` in its name is the update image that a running device downloads. Do not write it at address 0.

`BUILDINFO.txt` in each release lists the SHA-256 of each file.

## First setup

1. On the device, open **Settings** and select the **WiFi** row.
   - Two-button build: press Button 2 to open Settings.
   - One button only (the round board uses its BOOT button): hold Button 1 for 5 seconds or more to open Settings. Then hold it for 2 seconds or more to go to the next row, until the WiFi row.
2. The device opens the Wi-Fi network `AirSensor-Config`. Connect a phone or a computer to it and open `http://192.168.4.1`.
3. Type the 6-digit PIN that the device shows.
4. Type your Wi-Fi name, your Wi-Fi password and the address of your MQTT broker, then save.

The device tests the Wi-Fi settings before it saves them. If the test is good, the device restarts and connects. The setup page closes by itself after 5 minutes without use or after 5 wrong PINs; no stored setting changes in those cases.

A factory reset is on the same setup page.

## MQTT and Home Assistant

`<id>` is the last 6 hexadecimal characters of the device MAC address, in lower case. To find it, look in your broker for a new topic below `airsensor/device/`.

| Topic | Content |
|---|---|
| `airsensor/device/<id>/data` | Measured values, JSON |
| `airsensor/device/<id>/status` | Device status |
| `airsensor/device/<id>/settings` | Current settings |
| `airsensor/device/<id>/ota` | Update status |
| `airsensor/device/<id>/cmd/+` | Commands to the device (retained) |

Home Assistant finds the device by itself if MQTT discovery is on (topic prefix `homeassistant`).

## Updates

A device with Wi-Fi looks for a new release every 6 hours. If it finds a different version for its board, it downloads the image over HTTPS, installs it and restarts. If the new version does not start correctly, the device goes back to the version before.

Turn automatic updates off or on with the **auto-update** setting in Home Assistant.

You can also update with the cable at any time: flash the factory image of a newer release. This erases the stored settings, so do the first setup again.

## Problems and questions

Open an [issue](https://github.com/nazbert/airsensor/issues). Include the board, the firmware version from the Status page and what you see on the display.

## Licence

The firmware images are released under the MIT licence, see [LICENSE](LICENSE). They contain third-party software; the notices are in [THIRD_PARTY_NOTICES](THIRD_PARTY_NOTICES).

The firmware comes with no warranty. The sensor values are for information only. Do not use the device as a safety device or a medical device.
