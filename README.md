# FrSkySportTelemetry
This is a fork of pawelsky's FrSky library. The text of the orignal [RCGroups post](https://www.rcgroups.com/forums/showthread.php?2245978-FrSky-S-Port-telemetry-library-easy-to-use-and-configurable) is repeated below for safe keeping (with some markdown formatting)

___

## FrSky S-Port telemetry library - easy to use and configurable

Hi,

For those having Tarains radios or other system capable of receiving FrSky S-Port telemetry data I've created a library, that allows to emulate and/or decode the FrSky S-Port sensors or using Arduino compatible Teensy 3.x/4.x/LC board, ESP8266 or 5V/16MHz ATmega328P/ATmega2560 based boards (e.g. ProMini, Nano, Uno, Mega).

If you still use old FrSky telemetry, there is also a library for that as well. You can find it [here](https://www.rcgroups.com/forums/showthread.php?t=2465555)

The library initially created to work together with the [NazaDecoder](https://www.rcgroups.com/forums/showthread.php?t=1995704) or [NazaCanDecoder](https://www.rcgroups.com/forums/showthread.php?t=2071772) libraries that read telemetry data coming out of DJI Naza Lite/v1/v2 controllers (examples can be found in [post #36](https://www.rcgroups.com/forums/showpost.php?p=29662866&postcount=36)), but it can be used for other purposes as well.

The main goals that I had when creating this library was to have it:
1) easy to use
2) easy to configure
3) easy to add new sensors

The library consists of 5 main classes:
1. FrSkySportTelemetry - which is responsible for sending the S.Port data via one of the Teensy serial ports or SoftwareSerial ports on ESP8266 and ATmega328P/ATmega2560 based borads.
2. FrSkySportDecoder - which is responsible for decoding the data coming from the S.Port line via one of the Teensy serial ports or SoftwareSerial ports on ESP8266 and ATmega328P/ATmega2560 based borads.
3. FrSkySportPolling - which is a base class for emulating data polling (useful when there is no device actively polling the S.Port data such as the receiver). There are following two polling methods available:
    * FrSkySportPollingSimple - which simply polls each sensor one after another in a loop (default)
    * FrSkySportPollingDynamic - which polls responding sensors more often (just like FrSky receivers do)
4. FrSkySportSingleWireSerial - which is responsible for S-Port-like single wire serial transmission.
5. FrSkySportSensor - which is a base class for all the sensors. So far following sensors have been implemented (with the exception of the last one all of them can both emulate and decode the data):
    * FrSkySportSensorAss - which emulates [ASS-70/ASS-100](https://www.frsky-rc.com/product/ass-100/) airspeed sensors
    * FrSkySportSensorEsc - which emulates [Neuron ESC sensors](https://www.frsky-rc.com/frsky-neuron-40-60-80-esc/) (including SBEC)
    * FrSkySportSensorFcs - which emulates [FCS-40A/FCS-150A](https://www.frsky-rc.com/product/fas40s/) current sensors
    * FrSkySportSensotFlvss - which emulates [FLVSS/MLVSS](https://www.frsky-rc.com/product/mlvss/) LiPo voltage monitor sensor
    * FrskySportSensorGasSuite - which emulates the [Gas Suite](https://www.frsky-rc.com/product/gas-suite/) sensor
    * FrskySportSensorGps - which emulates the [GPS v2](https://www.frsky-rc.com/product/gps-2/) sensor
    * FrskySportSensorRpm - which emulates the [RPM/temperature](https://www.frsky-rc.com/product/rpm/) sensor
    * FrSkySportSensorSp2uart - which emulates the [S.Port to UART Converter type B](https://www.frsky-rc.com/product/sp2uartconverter/) sensor (analog ADC3/ADC4 inputs only)
    * FrskySportSensorVario - which emulates the [high precision variometer](https://www.frsky-rc.com/product/vari-h/) sensor
    * FrskySportSensorXjt- which is a special sensor that is only capable of decoding the additional (i.e. other than from the above sensors) data stream coming from the XJT transmitter module's S.Port. This data includes the old type (hub) telemetry and the special data such as ADC1, ADC2, SWR, RSSI and RxBatt.

The library is very easy to use, you can define which sensors to include, you can change their default IDs (e.g. to avoid conflicts or to use 2 FLVSS sensors to 12S batteries). The library makes sure that the sensors respond to correct ID being polled/transmitted (not just any ID or a fixed list of IDs as in some of the existing libraries) so you can even mix the virtual sensors with the real ones without conflicts. You can also define which of the Teensy serial ports to use. In fact you can send/decode different data on different ports by creating multiple telemetry objects.

The library can be used for both emulating the S.Port sensors and decoding the S.Port data, *but never at the same time!*

Attached you'll find the library itself and a picture showing the encoder code in action. As you can see the data displayed for FCS and FLVSS sensor matches the data set in code.

This library is for __non-commercial use only__, and no, __I do not plan to put this code on Github__

## ENCODER - emulating the S.Port sensor

Here is a quick example on how the library is used to emulate a sensor

```C
#include "FrSkySportSensor.h"
#include "FrSkySportSensorFlvss.h"
#include "FrSkySportSensorFcs.h"
#include "FrSkySportSingleWireSerial.h"
#include "FrSkySportTelemetry.h"

FrSkySportSensorFlvss flvss1;
FrSkySportSensorFlvss flvss2(FrSkySportSensor::ID15);
FrSkySportSensorFcs fcs;
FrSkySportTelemetry telemetry;

void setup()
{
  telemetry.begin(FrSkySportSingleWireSerial::SERIAL_3, &flvss1, &flvss2, &fcs);
}

void loop()
{

  /* DO YOUR STUFF HERE */
  /* Make sure you do not do any blocking calls in the loop, e.g. delay()!!! */

  flvss1.setData(4.10, 4.11, 4.12, 4.13, 4.14, 4.15);
  flvss2.setData(3.10, 3.11, 3.12, 3.13, 3.14, 3.15);
  fcs.setData(25.3, 12.6);
  telemetry.send();
}
```

As you can see it is super simple, all you need to do is to:
1) create the sensors you want to use and the telemetry object
2) call the begin method of the telemetry object defining the port and pointers to used sensors (up to 28)
3) update the sensor data when necessary
4) call the send method of the telemetry object periodically (i.e. within milliseconds from previous call)

Sensors will use their default physical IDs, but this can be changed by specifying different ID when the sensor object is created (see flvss2 sensor in the example above).

__Make sure you DO NOT BLOCK THE LOOP (milliseconds matter) with calls to methods such as delay() or other time consuming calls to other libraries!!!__

For simplicity it is not possible to remotely change data send frequency which every sensor has defined, the values are as specified by FrSky.

Have a look at the FrSkySportTelemetryExample included in the library to have a better understanding on what parameters can be set for each of the sensors.


## DECODER - decoding the S.Port data

Here is a quick example on how the library is used to decode a S.Port Sensor data.

```C
#include "FrSkySportSensor.h"
#include "FrSkySportSensorFlvss.h"
#include "FrSkySportSensorFcs.h"
#include "FrSkySportSingleWireSerial.h"
#include "FrSkySportDecoder.h"

FrSkySportSensorFlvss flvss1;
FrSkySportSensorFlvss flvss2(FrSkySportSensor::ID15);
FrSkySportSensorFcs fcs;
FrSkySportDecoder decoder;

void setup()
{
  decoder.begin(FrSkySportSingleWireSerial::SERIAL_3, &flvss1, &flvss2, &fcs);
}

void loop()
{
  // Call this on every loop
  decoder.decode();

  // Make sure that all the operations below are short or call them periodically otherwise you'll be losing telemetry data
  flvss1.getCell1();  // Read cell 1 voltage from the standard FLVSS sensor and do something with it
  flvss2.getCell2();  // Read cell 2 voltage from the ID15 FLVSS sensor and do something with it
  fcs.getCurrent();  // Read current from the fcs sensor and do something with it

  /* DO YOUR STUFF HERE */
  /* Make sure you do not do any blocking calls in the loop, e.g. delay()!!! */
}
```

As you can see it is super simple, all you need to do is to:
1) create the sensors you want to use and the decoder object
2) call the begin method of the decoder object defining the port and pointers to used sensors (up to 28)
4) call the decode method of the decoder object on every loop
3) read the data from sensors and do something with it (i.e. within milliseconds from previous call)

Sensors will use their default physical IDs, but this can be changed by specifying different ID when the sensor object is created (see flvss2 sensor in the example above). When you want the sensor data to be decoded regardless of which physical ID it comes from use the special __FrSkySportSensor::ID_IGNORE ID__, but be aware that in this case you won't be able to differentiate which particular sensor this data came from.

__Make sure you DO NOT BLOCK THE LOOP (milliseconds matter) with calls to methods such as delay() or other time consuming calls to other libraries!!!__

You can decode data from real sensors, my telemetry library, from [Taranis serial port](https://github.com/opentx/opentx/wiki/Taranis-serial-port) that can be found in the battery compartment (make sure it is configured to mirror S.Port data in radio menu)or S-Port connector in the Taranis JR TX module bay.

You'll find more detailed example in the FrSkySportDecoderExample and FrSkySportXjtDecoderExample directory.

## Data polling

To send the data the S.Port sensors must be actively polled. Normally that is done by the receiver that the sensor is connected to. If for whatever reason you decide to use this library in a configuration that does not have the receiver (or any other device actively polling) in the S.Port chain you have to enable library's internal polling mechanisms. The library offers two types of polling:
* __simple__ (legacy) implemented by class FrSkySportPollingSimple, where each sensor is polled one after another in a loop. This means that regardless of whether the sensor is active or not it will have to wait its turn to report until the whole loop repeats
* __dynamic__ (FrSky-like) implemented by class FrSkySportPollingDynamic, where active sensors are polled more often , without waiting for all the inactive ones, improving the data refresh rate. This is similar to how the original FrSky receivers do polling, e.g. if there is one active sensor it will be polled every 24ms instead of 336ms, when there are two active sensors they will be polled every 36ms instead of 336ms, etc.

Table below shows an example comparison on how sensors are polled with Simple and Dynamic polling when sensors ID1, ID3 and ID4 are responding (marked with asterisk *)

 Time  | Simple | Dynamic
-------|--------|---------
 12ms  |  ID1*  |  ID1*
 24ms  |  ID2   |  ID1*
 36ms  |  ID3*  |  ID2
 48ms  |  ID4*  |  ID1*
 60ms  |  ID5   |  ID3*
 72ms  |  ID6   |  ID1*
 84ms  |  ID7   |  ID3*
 96ms  |  ID8   |  ID4*
 108ms |  ID9   |  ID1*
 120ms |  ID10  |  ID3*
 132ms |  ID11  |  ID4*
 144ms |  ID12  |  ID5 
 156ms |  ID13  |  ID1*
 168ms |  ID14  |  ID3*
 180ms |  ID15  |  ID4*
 192ms |  ID16  |  ID6
 204ms |  ID17  |  ID1*
 216ms |  ID18  |  ID3*
 228ms |  ID19  |  ID4*
 240ms |  ID20  |  ID7
 252ms |  ID21  |  ID1*
 264ms |  ID22  |  ID3*
 276ms |  ID23  |  ID4*
 288ms |  ID24  |  ID8
 300ms |  ID25  |  ID1*
 312ms |  ID26  |  ID3*
 324ms |  ID27  |  ID4*
 336ms |  ID28  |  ID9
 348ms |  ID1*  |  ID1*
  ...  |  ...   |  ...

       
To enable polling you need to instantiate the FrSkySportTelemetry or FrSkySportDecoder class object passing the instance of and __FrSkySportPollingSimple__ or __FrSkySportPollingDynamic__ class object as a parameter. If you don't do that NULL (polling disabled) will be used as a default. Don't forget to #include an appropriate header file.

```FrSkySportTelemetry telemetry(new FrSkySportPollingDynamic()); // Create telemetry object with dynamic polling```

```FrSkySportDecoder decoder(new FrSkySportPollingSimple()); // Create decoder object with simple polling```

In examples, to make things easier polling can be enabled by uncommenting the following line
```//#define POLLING_ENABLED```

For the telemetry to be properly decoded on the Taranis radio RSSI data is required, so when polling is enabled the library also sends out RSSI data at regular intervals, accompanied by the RxBatt data (both can be set by the user)

__As both encoder and decoder have the polling capability, if you use both in your setup make sure that only one or the other has the polling enabled, not both at the same time.__


## Connections

Below you can also find attached connection diagrams for both encoder and decoder (note that they are different). Pick one of the Teensy 3.x/4.x/LC serial TX or 328P based board 2-12 pins (one that is not used for other purposes) and tell the library to use it in telemetry.begin or decoder.begin function. Depending on the board used you should chose (NOTE: SERIAL_x and SOFT_SERIAL_PIN_x means inverted, single-wire connection, while SERIAL_x_EXTINV means uninverted two-wire connection and expects signal to be inverted externally):
  * __SERIAL_1, SERIAL_2, SERIAL_3, SERIAL_1_EXTINV, SERIAL_2_EXTINV or SERIAL_3_EXTINV__ for Teensy 3.0/3.1/3.2/LC boards (note that the additional SERIAL_USB shall only be used for debug purposes if you know what you are doing)
  * __SERIAL_1, SERIAL_2, SERIAL_3, SERIAL_4, SERIAL_5, SERIAL_6, SERIAL_1_EXTINV, SERIAL_2_EXTINV, SERIAL_3_EXTINV, SERIAL_4_EXTINV, SERIAL_5_EXTINV or SERIAL_6_EXTINV__ for Teensy 3.5/3.6 boards (note that the additional SERIAL_USB shall only be used for debug purposes if you know what you are doing)
  * __SERIAL_1, SERIAL_2, SERIAL_3, SERIAL_4, SERIAL_5, SERIAL_6, SERIAL_7, SERIAL_1_EXTINV, SERIAL_2_EXTINV, SERIAL_3_EXTINV, SERIAL_4_EXTINV, SERIAL_5_EXTINV, SERIAL_6_EXTINV or SERIAL_7_EXTINV__ for Teensy 4.0/4.1 board (note that the additional SERIAL_USB shall only be used for debug purposes if you know what you are doing). Note that for Teensy 4.1 there is an additional Serial8 available for which you can use __SERIAL_8 or SERIAL_8_EXTINV__
 * __SOFT_SERIAL_PIN_4/SOFT_SERIAL_PIN_D2, SOFT_SERIAL_PIN_5/SOFT_SERIAL_PIN_D1, SOFT_SERIAL_PIN_12/SOFT_SERIAL_PIN_D6, SOFT_SERIAL_PIN_13/SOFT_SERIAL_PIN_D7, SOFT_SERIAL_PIN_14/SOFT_SERIAL_PIN_D5, SOFT_SERIAL_PIN_15/SOFT_SERIAL_PIN_D8 or SERIAL_EXTINV__ for ESP8266 based boards
  * __SOFT_SERIAL_PIN_2 to SOFT_SERIAL_PIN_12 or SERIAL_EXTINV__ for ATmega328P based boards (e.g. Pro Mini, Nano, Uno)
  * __SOFT_SERIAL_PIN_10, SOFT_SERIAL_PIN_11, SOFT_SERIAL_PIN_12, SOFT_SERIAL_PIN_13, SOFT_SERIAL_PIN_14, SOFT_SERIAL_PIN_15, SOFT_SERIAL_PIN_50, SOFT_SERIAL_PIN_51, SOFT_SERIAL_PIN_52, SOFT_SERIAL_PIN_53, SERIAL_EXTINV, SERIAL_1_EXTINV, SERIAL_2_EXTINV or SERIAL_3_EXTINV__ for ATmega2560 based boards (e.g. Mega)

Note that for ESP8266 and ATmega328P/ATmega2560 based boards you need to include the SoftwareSerial library in your main sketch: ```#include "SoftwareSerial.h"```

__There were reports that for some reason the library does not work when used with 1.6.0 version of Arduino IDE. This is most likely due to the changed toolchain and modified timings. If you experience problems please upgrade to a newer version where the timing calculation has been modified ([1.6.3 has been confirmed working](https://www.rcgroups.com/forums/showpost.php?p=31365392&postcount=190)) - that shall help.__

Also note that to avoid interrupt conflict with the mentioned SoftwareSerial library you need to disable attitude (roll/pitch) sensing in the NazaDecoder library by uncommenting the following line in NazaDecoder.h file: ```//#define ATTITUDE_SENSING_DISABLED```

__Make sure you understand Teensy 3.x/4.x/LC/ESP8266/ATmega328P/ATmega2560 based board powering options before choosing how to power your board!__

## Supported Application IDs and defaults

The table below shows all Application IDs supported by the sensor classes (except FrskySportSensorXjt), together with defualt Physical IDs and data poll times

| Sensor class             | Phys. ID | App ID                                                                 | #define                                                                                  | Data type                                                                 | Poll time [ms]                 |
|--------------------------|----------|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|--------------------------------|
| FrSkySportSensorAss      | ID10     | 0x0A00                                                                 | ASS_SPEED_DATA_ID                                                                         | air speed                                                                 | 500                            |
| FrSkySportSensorEsc      | ID17     | 0x0B50<br>0x0B60<br>0x0B70<br>0x0E50                                   | ESC_POWER_DATA_ID<br>ESC_RPM_CONS_DATA_ID<br>ESC_TEMP_DATA_ID<br>ESC_SBEC_DATA_ID         | ESC voltage/current<br>RPM/consumption<br>temperature<br>SBEC voltage/current | 300<br>300<br>300<br>300        |
| FrSkySportSensorFcs      | ID3      | 0x0200<br>0x0210                                                       | FCS_CURR_DATA_ID<br>FCS_VOLT_DATA_ID                                                       | current<br>voltage                                                      | 500<br>500                    |
| FrSkySportSensorFlvss    | ID2      | 0x0300                                                                 | FLVSS_CELL_DATA_ID                                                                        | cell voltage                                                             | 300                            |
| FrskySportSensorGasSuite | ID23     | 0x0D00<br>0x0D10<br>0x0D20<br>0x0D30<br>0x0D40<br>0x0D50<br>0x0D60<br>0x0D70 | GAS_SUITE_T1_DATA_ID<br>GAS_SUITE_T2_DATA_ID<br>GAS_SUITE_RPM_DATA_ID<br>GAS_SUITE_RES_VOLUME_DATA_ID<br>GAS_SUITE_RES_PERCENT_DATA_ID<br>GAS_SUITE_FLOW_DATA_ID<br>GAS_SUITE_FLOW_MAX_DATA_ID<br>GAS_SUITE_FLOW_AVG_DATA_ID | temperature 1<br>temperature 2<br>RPM<br>residual volume<br>residual percent<br>flow<br>max flow<br>average flow | 100<br>100<br>100<br>100<br>100<br>100<br>100<br>100 |
| FrskySportSensorGps      | ID4      | 0x0800<br>0x0820<br>0x0830<br>0x0840<br>0x0850                         | GPS_LAT_LON_DATA_ID<br>GPS_ALT_DATA_ID<br>GPS_SPEED_DATA_ID<br>GPS_COG_DATA_ID<br>GPS_DATE_TIME_DATA_ID | latitude/longitude<br>altitude<br>ground speed<br>COG<br>date/time        | 1000<br>500<br>500<br>500<br>10000 |
| FrskySportSensorRpm      | ID5      | 0x0400<br>0x0410<br>0x0500                                             | RPM_T1_DATA_ID<br>RPM_T2_DATA_ID<br>RPM_ROT_DATA_ID                                       | temperature 1<br>temperature 2<br>RPM                                   | 1000<br>1000<br>500           |
| FrSkySportSensorSp2uart  | ID7      | 0x0900<br>0x0910                                                       | SP2UARTB_ADC3_DATA_ID<br>SP2UARTB_ADC4_DATA_ID                                             | ADC 3<br>ADC 4                                                         | 500<br>500                    |
| FrskySportSensorVario    | ID1      | 0x0100<br>0x0110                                                       | VARIO_ALT_DATA_ID<br>VARIO_VSI_DATA_ID                                                     | altitude<br>VSI                                                        | 200<br>100                    |



When creating my library I used some of the information that I found on this [webpage](https://code.google.com/p/telemetry-convert/wiki/FrSkySPortProtocol)

## Other projects using this library

Here are references to some other projects using this library. __I do not provide support for these - in case of problems contact their authors.__

* [Arduino Voltmeter compatible to FrSky SPort Telemetry with minimal additional hardware](https://www.rcgroups.com/forums/member.php?u=731873) by Dakkaron
* [FrSkySportTelemetry extension for INav](https://github.com/variostudio/FrSkySportTelemetry) by RedTroll
* [Arduino S.Port to MAVLink Converter](https://github.com/davwys/arduino-sport-to-mavlink) by Closus
* [DIY FrSky telemetry multi-sensor]( http://axelsdiy.brinkeby.se/?p=2009) by Axel Brinkeby


## FrSky S.Port Telemetry library changelog

| Version     | Changes |
|-------------|---------|
| 20250412 | [FIX] Fixed Serial2 and Serial4 not working correctly on Teensy 4.x<br>[FIX] Corrected #define POLLING_ENABLED in FrSkySportTelemetryExample.ino to be commented out by default<br>[FIX] Added some missing error messages if an unknown board is used |
| 20210509 | [NEW] Added Gas Suite sensor<br>[NEW] Simplified send function in sensors<br>[FIX] Removed unnecessary semicolon from GPS_DATE_TIME_DATA_PERIOD define |
| 20210110 | [FIX] Fixed unnecessary switching of TX/RX mode when serial with EVTINV is used |
| 20210108 | [FIX] Fixed detecting empty frame in decoder<br>[NEW] Added support for non-inverted two wire serial (e.g. external S.Port inverter, Jumper R8, Pixhawk)<br>[NEW] Added support for Teensy 4.1<br>[FIX] Refactored FrSkySportSingleWireSerial code for readability<br>[FIX] Minor editorial corrections |
| 20200503 | [NEW] Added FrSkySportPollingDynamic and FrSkySportPollingSimple polling classes<br>[NEW] Updated examples to use polling classes<br>[NEW] Telemetry send method now returns last sent APP ID<br>[NEW] Added support for ATmega2560 (Arduino Mega)<br>[FIX] Fixed typos<br>[FIX] Fixed cell voltage detection in FrSkySportSensorXjt<br>[FIX] Replaced deprecated boolean with bool in FrSkySportDecoder<br>[FIX] Added missing literals to keywords.txt<br>[FIX] Renamed CRC define to CRC_BYTE<br>[FIX] Added define for empty frame header |
| 20200112 | [NEW] Added ESC sensor and updated examples<br>[NEW] Added library.properties<br>[NEW] Added alternative Dxx pin naming for ESP8266<br>[FIX] Limited ESP8266 pins to supported SoftwareSerial pins |
| 20191110 | [FIX] Fixed incorrect detection of stuffed bytes in FrSkySportDecoder |
| 20191103 | [NEW] Added support for Teensy 4.0<br>[FIX] Rearranged serial config using defines instead of magic numbers<br>[FIX] Simplified Teensy detection |
| 20190824 | [NEW] Added support for ESP8266 boards |
| 20180402 | [NEW] Added RxBatt data when polling enabled<br>[NEW] Added RSSI/RxBatt setter methods<br>[NEW] Updated example<br>[FIX] Minor editorial corrections |
| 20171114 | [FIX] Removed unnecessary debug message from FrSkySportDecoder.cpp |
| 20160919 | [NEW] Added support for Teensy LC, 3.5, 3.6<br>[NEW] Added MLVSS note in FLVSS class |
| 20160818 | [NEW] Added simplified polling to telemetry and decoder classes<br>[NEW] Added POLLING_ENABLED define<br>[NEW] Unified connection diagrams |
| 20160324 | [NEW] Increased max sensors from 10 to 28 |
| 20160313 | [FIX] Added missing function names to keywords<br>[FIX] Renamed FrSkySportSensorTaranisLegacy to FrSkySportSensorXjt<br>[NEW] Added Xjt decoder example<br>[NEW] Added decoding for ADC1/ADC2/RSSI/SWR/RxBatt<br>[NEW] Added vertical speed decoding from FVAS |
| 20151108 | [FIX] Data ID returned only when full value decoded<br>[FIX] Fixed empty FLVSS message crash<br>[NEW] Added defines for decoded values<br>[NEW] Added keyword highlighting IDs<br>[NEW] Clarified FCS sensor differences |
| 20151020 | [FIX] Fixed CRC stuffing<br>[FIX] Fixed lat/lon calculation |
| 20151018 | [NEW] Added Taranis legacy decoder<br>[NEW] Added CRC checking<br>[NEW] Added return result to decode methods<br>[FIX] Removed redundant CRC calculations<br>[FIX] Simplified cell voltage decoding |
| 20151008 | [NEW] Added Decoder class and examples<br>[FIX] Fixed RPM sensor types and rounding<br>[FIX] Minor editorial corrections |
| 20150921 | [NEW] Added airspeed sensor |
| 20150725 | [NEW] Added transmission periods<br>[NEW] Added SP2UART sensor (ADC3/ADC4 only) |
| 20150319 | [FIX] Corrected 328p serial pin mode handling<br>[FIX] Recommended 4.7 kΩ resistor for S.Port protection |
| 20141129 | [FIX] Fixed GPS coordinate display on 328p |
| 20141120 | [NEW] Added support for 328P boards (Pro Mini, Nano, Uno)<br>[NEW] Added connection diagrams |
| 20140914 | Initial version of the library |
