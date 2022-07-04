# ModuleAir V2

More information on Aircarto.fr

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
* Gets measurements from a full range of sensors
* Transmits data with WiFi or LoRaWAN to different databases
* Gets AQ forecasts from the AtmoSud API, the official Institute for Air Quality in SE France
* Displays the measurements and the forecasts (French AQI, NO2, O3, PM10, PM2.5) on the matrix
* Fully configurable through a web interface

## Libraries
* bblanchon/ArduinoJson@6.18.3
* 2dom/PxMatrix LED MATRIX library@^1.8.2
* adafruit/Adafruit GFX Library@^1.10.12
* adafruit/Adafruit BusIO@^1.9.8
* https://github.com/IntarBV/MHZ16_uart
* https://github.com/WifWaf/MH-Z19.git
* sensirion/Sensirion Core@^0.6.0
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

##Flashing

Please use Platformio to flash the board.
The .ini file should be able to get all the needed boards, platforms and libraries from the internet

##Library changes

To force the use of both the SPIs on the ESP32, the SPI library and the PXMatrix librar has to be corrected a bit.

SPI.cpp

Modify as this:
`#if CONFIG_IDF_TARGET_ESP32
SPIClass SPI(VSPI);
SPIClass SPI_H(HSPI);
#else
SPIClass SPI(FSPI);
#endif`

SPI.h

Add this line at the bottom:
`extern SPIClass SPI_H;`

PxMatrix.h

Replace all `SPI` with `SPI_H`.

Verify that those pins are defined:

`// HW SPI PINS
#define SPI_BUS_CLK 14
#define SPI_BUS_MOSI 13
#define SPI_BUS_MISO 12
#define SPI_BUS_SS 4`

##Font changes

The default glcdfont.c of the Adafruit GFX library was modified to add new characters.
Copy the content of the glcdfont_mod.c file in the Fonts folder and paste it in the glcdfont.c file in the Adafruit GFX library in the folder of the choosed board in the .pio folder.

##Pin mapping

You can find the main pin mapping for each board in the ext_def.h file.
THe pin mapping for the LoRaWAN module is in the file moduleair.cpp under the Helium/TTN LoRaWAN comment.

##Configuration

The process is the same as for the Sensor.Community firmware.
On the first start, the sation won't find any known network and it will go in AP mode producing a moduleair-XXXXXXX network. Connect to it from a PC or a smartphone and open the address http://192.168.4.1.
If needed, the password is moduleaircfg.

For the WiFi connection: type your credentials
For the LoRaWAN connection : type the APPEUI, DEVEUI and APPKEY as created in the Helium or TTN console.

Choose the sensors, the displays and the API in the different tabs. For coding reason, it was not possible to use radios for the PM sensors and the CO2 sensors. Please don't check both sensors of the same type to avoid problems…

Please don't decrease the measuring interval to spare connection time.

If the checkbox "WiFi transmission" is not checked, the station will stay in AP mode for 10 minutes and then the LoRaWAN transmission will start. During those 10 minutes or after a restart, you can change the configuration.

If the checkbox "WiFi transmission" is checked, the sensor will be always accessible through your router with an IP addess : 192.168.<0 or more>.<100, 101, 102…>. In that case the data streams will use WiFi and not LoRaWAN (even if it is checked).

## LoRaWAN payload
The payload consists in a 30 bytes (declared as a 31 according to the LMIC library) array.

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

