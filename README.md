# ModuleAir V2

Mpre information on Aircarto.fr

## Supported sensors
* Nova PM SDS011
* Groupe Tera NextPM
* SGP40
* MH-Z16
* MH-Z19
* BME280

## Display
* 64x32 RGB Matrix
* OLED SSD1306

## Features
* Gets measurement from a full range of sensors
* Transmits data with WiFi or LoRaWAN to different databases
* Gets AQ forecasts from the AtmoSud API, the official Institute for Air Quality in SW France
* Displays the measurement and the forecasts on the matrix
* Fully configurable through a web interface

## Libraries
* bblanchon/ArduinoJson@6.18.3
* 2dom/PxMatrix LED MATRIX library@^1.8.2
* adafruit/Adafruit GFX Library@^1.10.12
* adafruit/Adafruit BusIO@^1.9.8
* https://github.com/IntarBV/MHZ16_uart
* https://github.com/WifWaf/MH-Z19.git
* sensirion/Sensirion I2C SGP40@^0.1.0
* mcci-catena/MCCI LoRaWAN LMIC library@^4.1.1
* ThingPulse/ESP8266 and ESP32 OLED driver for SSD1306 displays @ ^4.2.1

And the ESP32 platform librairies:
* Wire
* WiFi
* DNSServer
* WiFiClientSecure
* HTTPClient
* FS
* SPIFFS
* WebServer
* Update
* ESPmDNS

## Boards
The code is developped on a ESP32 DevC but other boards should be supported:
* ESP32 DOIT DevKit V1
* ESP32 Mini S2
* Heltec Wifi Lora 32 V2
* TTGO Lora 32 V2.1 Paxcounter

## LoRaWAN payload
The payload consists in a 31 bytes array.

The value are initialised for the first uplink at the end of the void setup() which is send according to the LMIC library examples.

0x00, config = 0 (see below)\
0xff, 0xff, sds PM10 = -1\
0xff, 0xff, sds PM2.5 = -1\
0xff, 0xff, npm PM10 = -1\
0xff, 0xff, npm PM2.5 = -1\
0xff, 0xff, npm PM1 = -1\
0xff, 0xff, mhz16 CO2 = -1\
0xff, 0xff, mhz19 CO2 -1\
0xff, 0xff, sgp40 OVC = -1\
0x80, bme temp = -128\
0xff, bme rh = -1\
0xff, 0xff, bme press = -1\
0x00, 0x00, 0x00, 0x00, lat = 0.0 (as float)\
0x00, 0x00, 0x00, 0x00, lon = 0.0 (as float)\
0xff sel = -1 (see below)\

Those default values will be replaced during the normal use of the station according to the selected sensors.

The fist byte is the configuation summary, representeed as an array of 0/1 for false/true:

configlorawan[0] = cfg::sds_read;\
configlorawan[1] = cfg::npm_read;\
configlorawan[2] = cfg::bmx280_read;\
configlorawan[3] = cfg::mhz16_read;\
configlorawan[4] = cfg::mhz19_read;\
configlorawan[5] = cfg::sgp40_read;\
configlorawan[6] = cfg::display_forecast;\
configlorawan[7] = cfg::has_wifi;\

It produce single byte which will have to be decoded on server side.

For example:

10101110 (binary) = 0xAE (hexbyte) =174 (decimal)
The station as a SDS011, a BME280, a MH-Z19, a SGP40, the forecast are activated, the WiFi is not activated.

The LoRaWAN server has to get the forecast data and transmit by downlink. Because the WiFi is not activated the uplink sensor data has to be sent to the databases.

If wifi is activated it is useless to decode the uplinks and transmit some downlinks because everything is already done though API calls and POST requests.

The last byte is a selector which tells the LoRaWAN server which kind of downlink values it should transmit (AQ index, NO2, O3, PM10, PM2.5 from the AtmoSud API). 5 downlinks will be sent each day.

