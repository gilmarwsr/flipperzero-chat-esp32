# FlipperZero SubGhz Chat Bridge

This project will allow you to communicate with [FlipperZero](https://flipperzero.one/)'s SubGhz chat feature using an ESP32 and a CC1101.

The chat bridge provides a web interface allowing multiple user's with different nicknames to connect and communicate with nearby flipper devices.

In order to build this you will need:

 * ESP32 - https://www.amazon.com/gp/product/B09GK74F7N/
 * CC1101 - https://www.amazon.com/gp/product/B01DS1WUEQ/
 * Computer with Arduino 1.8.18
     * ESP32 library support 2.0.5 (https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json)
     * SmartRC-CC1101-Driver-Lib (https://github.com/LSatan/SmartRC-CC1101-Driver-Lib)
     * Ability to upload FatFS image (https://github.com/lorol/arduino-esp32fs-plugin)
     * ArduinoJson 6.19.3 (https://arduinojson.org/?utm_source=meta&utm_medium=library.properties)
     * Adafruit_SSD1306 2.5.1 - https://github.com/adafruit/Adafruit_SSD1306


### Building your own bridge

 ESP-32 | CC1101
 -------|------------
 3.3v   | VIN
 GND    | GND
 5      | CS
 18     | SCLK
 19     | MISO / SO
 23     | MOSI / SI

Please note: The GDO0 and GDO2 pins are left unconnected.

Optionally you can add a screen using an SSD1306 OLED module.
https://www.amazon.com/gp/product/B0833PF7ML/

 ESP-32 | SSD1306
 -------|--------
 3.3v   | VIN
 GND    | GND
 GPIO21 | SDA
 GPIO22 | SCL

![Flipper Chat Module](https://openstatic.org/img/flipperchatmodule.png)

Once you've downloaded the source and properly setup your IDE for flashing:
  1. copy "base-settings.json" to "data/settings.json"
  2. edit the settings.json file in the data directory. This will be loaded at startup.

```json
{
  "startFrequency": 433.92,
    "apMode": true,
    "captiveDNS": false,
    "apSSID": "FlipperChat",
    "apPassword": "changeme",
    "wifi": [
        {
          "ssid": "Your network name",
          "password": "wifi pass"
        }
    ]
}

```
if "apMode" is set to true the ESP32 will create a network with the settings labeled "apSSID" and "apPassword". Otherwise please add the wifi networks that should be searched for to the json array named "wifi"

The "captiveDNS" setting will redirect any DNS queries to the device's apMode IP (10.10.10.1)

Flash Settings for ESP32 (in aruino ide)
 * CPU Frequency - 240Mhz
 * Flash Frequency - 80Mhz
 * Partition Scheme - Default 4MB with FFAT (1.2MB / 1.5MB)
 * Flash Mode - DIO
 * PSRAM - Disabled
 * Events Run on - Core 0
 * Arduino Runs on - Core 1

Now you are all set to flash the sketch and upload the FatFS image.

### Using the web interface

Navigate a web browser to the ESP32's ip (output on the serial console) and you will be prompted to enter a nickname. Multiple users and computers are able to connect to the same ESP32 with distinct nicknames, they will see each others messages, as well as flipper users.

Technically you could build two bridges and they would be able to communicate with each other as well.

The catch is the CC1101 can only tune into one frequency, so everyone must be on the same frequency and if someone changes the frequency it effects all users on that bridge. You can see the current tuned frequency in the upper left hand corner.

At any point while in the chat screen a connected user may type "/freq 433.92" to change the frequency, the new frequency must be expressed in Mhz.

### Websocket protocol

If you would like to make your own client to interface with this device the protocol is pretty simple. port 81 is a websocket server all messages are single line json objects.

Connect to ws://[deviceip]:81/

The server will immediately send a message like..
```json
{"clientIp": "192.168.34.131", "mhz": 315, "users": [{"u": "Lyp1n1", "s": "radio", "r": -36}]}
```

  * clientIp will be your ip as the device sees it
  * mhz is the current radio frequency
  * users is an array of json objects with three fields, u = username, s = source (radio/ip), r = rssi

To register a user send
```json
{"event":"join", "username":"Hacker5091"}
```

there should be a response like:
```json
{"event":"join", "username": "Hacker5091", "rssi": -63, "source": "192.168.0.131"}
```

To send a chat messge send
```json
{"event":"chat", "username": "Hacker5091", "text": "This is a chat message"}
```

To unregister a user
```json
{"event":"part", "username":"Hacker5091"}
```

To change the frequency
```json
{"event":"frequency", "mhz": 315, "username": "Hacker5091"}
```
