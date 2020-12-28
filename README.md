# M5Stack-LoRa-868-Network-Tester

A LoRaWAN Network Tester based on the M5Stack for the LoRa 868 module, compatible with TTN (The Things Network)

## Setup
The tester is designed to work with the following hardware:
  - M5Core (Basic, Gray or Fire)
  - M5Go Base (optional)
  - M5Stack GPS Module (optional)
  - [M5Stack LoRa 868 Module]

#### Required Libraries!
  - [M5Stack]
  - [TinyGPSPlus]
  - [NeoPixelBus]
  - [M5_UI]
  - [MCCI Arduino LMIC] - this Fork enables the LinkCheckRequest to get info on gateway number and margin from the backend

 
#### Installation and Configuration
Upload this sketch to your M5 using the Arduino IDE. M5Stack Fire users have to disable PSRAM, because it will interfer with UART2.
UART2 with GPIO 16 and 17 willbe used for the GPS module.

Edit the lmic_project_config.h

  - #define CFG_eu868 1
  - #define CFG_sx1276_radio 1
  - #define LMIC_USE_INTERRUPTS
    
By commenting out #define M5go it is possible to disable the M5GO Base. This will disable all NeoPixel related code and features.
By commenting out #define M5gps ist is possible to disable the M5GPS module. This will disable all GPS related code and features. 
  
Change the your TTN keys under //LoRaWAN ABP //LoRaWAN OTAA in the networktester.ino file. They are filled in as is from the TTN console. If you want yo use OTAA you have to register a second device for your application. 

Payload Decoder for TTN (also compatible with TTN Mapper integration):

```
function Decoder(b, port) {

  var lat = (b[0] | b[1]<<8 | b[2]<<16 | (b[2] & 0x80 ? 0xFF<<24 : 0)) / 10000;
  var lon = (b[3] | b[4]<<8 | b[5]<<16 | (b[5] & 0x80 ? 0xFF<<24 : 0)) / 10000;
  var alt = (b[6] | b[7]<<8 | (b[7] & 0x80 ? 0xFF<<16 : 0)) / 100;
  var hdop = b[8] / 10;

  return {
    
      latitude: lat,
      longitude: lon,
      altitude: alt,
      hdop: hdop
    
  };
}
```

## Instructions for Use

#### Menu

On boot you will be presented with the "Boot-Logo" followed by the first working mode. At the moment the tester has 7 modes to select:
  - [NACK](#nack) 
  - [ACK](#ack)  
  - [MAN](#man)  
  - [LCM](#lcm)
  - [SSV](#ssv)
  - [OTAA](#otaa)
  - [SET](#set)
 
You can move between menu items by pushing the button A. 

![Menu Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/menu.jpg "Fig 1. Menu")
  
#### NACK 
#### (No Acknowladge)
"NACK" is a mode that utlises the current device [settings](#set) to perform periodic transmissions. "NACK" mode is great for use with TTN Mapper.

![NACK Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/nack.jpg "Fig 2. NACK")

Pushing button B will let you cycle through each spreadfactor. By pushig button C the display and LEDs will be turned off. Pushing button C again will turn them on.

#### ACK 
#### (Acknowladge)
"ACK" will perform the same test as NACK but it will request an ACK for every transmission. The RSSI and SNR values of the received packet will be shown on the display. Pushing button B will let you cycle through each spreadfactor. By pushig button C the display and LEDs will be turned off. Pushing button C again will turn them on.

![ACK Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/ack.jpg "Fig 3. ACK")

#### MAN 
#### (Manual)
"MAN" will send a LoRaWAN packet with ACK by pushing button C. Pushing button B will let you cycle through each spreadfactor.

![MAN Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/man.jpg "Fig 4. MAN")

#### LCM 
#### (LinkCheckMode)
"LCM" is a mode that will trigger a LinkCheckRequest. The TTN backend will report back the number of gateways which received the request. Pushing button B will let you cycle through each spreadfactor. The request is triggered by button C.

![LCM Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/lcm.jpg "Fig 5. LCM")

#### SSV 
#### (SiteSurvey)
"SSV" is supposed as mode for testing a location. During SSV mode the DutyCycle check will be disabled an the Node will send a LinkCheckRequest for every spreadfactor from SF7 to SF12. After the test the node will show you on which datarates a ACK was received back. The data is also stored on the SD card in GeoJSON format an could be analyzed with [geojson.io]

![SSV Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/ssv-1.jpg "Fig 6. SSV running")
![SSV Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/ssv-2.jpg "Fig 7. SSV results")

#### OTAA 
#### (OverTheAirActivation)
"OTAA" enables the tester to perform OTAA-Joins. By selecting Join the tester will try to join the TTN Network. After an successful the the tester will start with periodic transmissions. You have the choice between transmission with or without ACK. The RSSI and SNR values of the received packet will be shown on the display. If there is no valid GPS fix, a packet can by manually send by pushing button B.
Firmware version 2.1.19 of the LoRaWAN module is mandatory for OTAA to work.

![OTAA Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/otaa.jpg "Fig 7. OTAA")

#### SET 
#### (Settings)

"SET" allows to change the transmission intervall in NACK or ACK mode. Possible settings are 15/30/45/60/120 seconds. Pressing button C will active the powersaving mode. The node will go to deep sleep and wakes up according to the transmission intervall. Sleep mode can only be stopped by resetting the devicde.

![SET Image](https://github.com/Bjoerns-TB/M5Stack-LoRaWAN-Network-Tester/blob/master/images/set.jpg "Fig 7. SET")

## Notes
  - The DutyCycle check ist activated except for the SSV mode.
  - If you have a valid GPS fix the GPS track will be written to the SD card as a GPX file.
  - Periodic transmission will only work with a valid GPS fix and an GPS age below 2 seconds
  
## Changelog

  - 30.12.2020
    - First commit


## ToDo
  - create tasks for M5UI
  - improve powersave features (GPS module)


[M5Stack]: https://github.com/m5stack/M5Stack
[TinyGPSPlus]: https://github.com/mikalhart/TinyGPSPlus
[NeoPixelBus]: https://github.com/Makuna/NeoPixelBus
[M5_UI]: https://github.com/dsiberia9s/M5_UI
[geojson.io]: http://geojson.io/
[M5Stack LoRa 868 Module]: https://m5stack.com/products/lora-module-868mhz
[MCCI Arduino LMIC]: https://github.com/Bjoerns-TB/arduino-lmic/tree/LMIC_setLinkCheckRequestOnce


