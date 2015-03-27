Arduino IDE with ESP8266 support
================================

This is an early beta. Things might not work as expected, please do report any bugs you find.

Downloads
---------

| OS | Build status | Latest release |
| --- | ------------ | -------------- |
| Linux | [![Linux build status](http://img.shields.io/travis/igrr/Arduino.svg)](https://travis-ci.org/igrr/Arduino) | [arduino-1.6.1-linux64.tar.xz](releases/download/arduino-1.6.1-esp8266-1/arduino-1.6.1-linux64.tar.xz) |
| Windows | [![Windows build status](http://img.shields.io/appveyor/ci/igrr/Arduino.svg)](https://ci.appveyor.com/project/igrr/Arduino) | [arduino-1.6.1-win32.zip](releases/download/arduino-1.6.1-esp8266-1/arduino-1.6.1-win32.zip) |
| OS X |  | [arduino-1.6.1-macosx.zip](releases/download/arduino-1.6.1-esp8266-1/arduino-1.6.1-macosx.zip) |


What works
----------

- **pinMode, digitalRead, digitalWrite**

	Pin numbers correspond directly to the esp8266 GPIO pin numbers. To read GPIO2,
	call digitalRead(2);
	GPIO0-GPIO15 can be INPUT, OUTPUT, INPUT_PULLUP, and OUTPUT_OPEN_DRAIN.
	GPIO16 can be INPUT or OUTPUT.

- **analogRead(0)** reads the value of the ADC channel connected to the TOUT pin.

- **pin interrupts (attachInterrupt, detachInterrupt)**

	Interrupts may be attached to any GPIO pin, except GPIO16. Standard Arduino interrupt 
	types are supported: CHANGE, RISING, FALLING.

- **shiftIn, shiftOut**
- **millis, micros**
- **delay, delayMicroseconds, yield**

	Remember that there is a lot of code that needs to run on the chip besides the sketch 
	when WiFi is connected. WiFi and TCP/IP libraries get a chance to handle any pending
	events each time the loop() function completes, OR when delay(...) is called.
	If you have a loop somewhere in your sketch that takes a lot of time (>50ms) without
	calling delay(), you might consider adding a call to delay function to keep the WiFi
	stack running smoothly.
	There is also a yield() function which is equivalent to delay(0). The delayMicroseconds
	function, on the other hand, does not yield to other tasks, so using it for delays
	more than 20 milliseconds is not recommended.

- **Serial**

	Only 8n1 is supported right now. By default the diagnostic output from WiFi 
	libraries is disabled when you call Serial.begin. To enable debug output again,
	call Serial.setDebugOutput(true);

- **Ticker**

	Library for calling functions repeatedly with a certain period. Two examples included.

- **EEPROM**

	This is a bit different from standard EEPROM class. You need to call EEPROM.begin(size)
	before you start reading or writing, size being the number of bytes you want to use.
	Size can be anywhere between 4 and 4096 bytes.
	EEPROM.write does not write to flash immediately, instead you must call EEPROM.commit()
	whenever you wish to save changes to flash. EEPROM.end() will also commit, and will
	release the RAM copy of EEPROM contents. 
	Three examples included.

- **I2C (Wire library)**

	Only master mode works, and I haven't tested if Wire.setClock gives correct frequency.
	Before using I2C, you need to set pins you will use for SDA and SCL by calling
	Wire.pins(int sda, int scl), i.e. Wire.pins(0, 2); on ESP-01.

- **OneWire (from https://www.pjrc.com/teensy/td_libs_OneWire.html)**

	Library was adapted to work with ESP8266 by including register definitions into OneWire.h
	Note that if you have OneWire library in your Arduino/libraries folder, it will be used
	instead of the one that comes with the Arduino IDE (this one).

- **WiFi(ESP8266WiFi library)**

	This is mostly similar to WiFi shield library. Differences include:
		* WiFi.mode(m): set mode to WIFI_AP, WIFI_STA, or WIFI_AP_STA.
		* call WiFi.softAP(ssid) to set up an open network
		* call WiFi.softAP(ssid, passphrase) to set up a WPA2-PSK network
		* WiFi.macAddress(mac) is for STA, WiFi.softAPmacAddress(mac) is for AP.
		* WiFi.localIP() is for STA, WiFi.softAPIP() is for AP.
		* WiFi.RSSI() doesn't work
		* WiFi.printDiag(Serial); will print out some diagnostic info
	WiFiServer, WiFiClient, and WiFiUDP behave mostly the same way as with WiFi shield library.
	Three samples are provided for this library.

- **mDNS responder (ESP8266mDNS library)**

	Allows the sketch to respond to multicast DNS queries for domain names like "foo.local".
	See attached example and library README file for details.

- Other libraries that don't rely on low-level access to AVR registers.

- Upload via serial port
	Select "esptool" as a programmer, and pick the correct serial port.
	You need to put ESP8266 into bootloader mode before uploading code (pull GPIO0 low and 
	toggle power).

What is not done yet
--------------------

- analogWrite (PWM). ESP8266 has only one hardware PWM source. It is not yet clear how to use it with analogWrite API. Software PWM is also an option, but apparently it causes issues with WiFi connectivity.
- pulseIn
- SPI. HSPI and bit-banging are two interfaces that will be supported.
- I2C slave mode
- Serial modes other than 8n1
- WiFi.RSSI. SDK doesn't seem to have an API to get RSSI for the current network. So far the only
	way to obtain RSSI is to disconnect, perform a scan, and get the RSSI value from there.
- Upload sketches via WiFi. Conceptually and technically simple, but need to figure out how to provide the best UX for this feature.
- Samples for all the libraries


License and credits
-------------------

Arduino IDE is licensed under GPL, with Arduino core libraries licensed under LGPL.

This build includes an xtensa gcc toolchain, which is also under GPL.

Espressif SDK included in this build is under Espressif Public License.

Esptool written by Christian Klippel is licensed under GPLv2. 
The source with my modfications is available on github: https://github.com/igrr/esptool-ck

ESP8266 port contributed by Ivan Grokhotkov, ivan@esp8266.com.


