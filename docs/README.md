# ProgrammableHub - A HomeSpan Project

ProgrammableHub is a working example of how to add a Web Server alongside of HomeSpan to create a Programmable Light Server Hub.  Built using the [HomeSpan](https://github.com/HomeSpan/HomeSpan) HomeKit Library, ProgrammableHub is designed to run on an ESP32 device as an Arduino sketch.  

Hardware used for this project:

* An ESP32 board, such as the [Adafruit HUZZAH32 – ESP32 Feather Board](https://www.adafruit.com/product/3405)
* One or more LEDs with current-limiting resistors

This sketch demonstrates a number of advanced HomeSpan techniques, including:

* Dynamically adding and deleting HomeKit Accessories
* Using the ESP32's Non-Volatile Storage (NVS) to save arbitrary data
* Implementing a standalone Web Server to operate alongside of HomeSpan

### Notes

* Rather than hardcode a fixed number of LED Accessories, this sketch dynamically creates up to 16 Dimmable LEDs at runtime based on data stored in NVS.
* The LED Accessories are named Light-01 through Light-16, but these can be changed to whatever you would like from the Home App on your iPhone or iMac.
* To aid in identifying each LED, the name is also copied as the Serial Number for each Accessory.
* A standalone Web Interface is used to allow users to specify which Lights are associated with which ESP32 pins, and whether the Light is dimmable.  This information is stored in the following structure:

```C++
struct {
  uint8_t pin=0;
  uint8_t dimmable=0;
} lightData[NLIGHTS];
```

* The above structure is stored as an NVS *blob* name *LIGHTDATA* in an NVS space name *LIGHTS*.
* Upon startup, the sketch creates a Bridge as Accessory #1, and then dynamically adds Accessories #2 through #17, for each Light-01 through Light-16 that has a defined ESP32 pin.
* Non-dimmable lights are implemented with just on/off functionality using `Characteristic::On()`.
* Dimmable lights are implemented with an additional brightness control using `Characteristic::Brightness()`.
* The name of the device's Web Interface is [homespanhub.local](http://homespanhub.local).  The normal device-ID suffix has been supressed using the method `homeSpan.setHostNameSuffix("")`
* The device must be rebooted (either manually or via the Web Interface) for updates to propogate to your Home App.
* If the Home App does not reflect your changes shortly after rebooting, try restarting the Home App.
* The Web Interface is implemented using the ESP32-Arduino standard WebServer Library, which is compatible with the WiFi/TCP stack used by HomeSpan.  Note that ESPAsyncWebServer requires a different TCP stack and cannot be used with HomeSpan.
* Notable configuration settings:
  * HomeSpan normally uses port 80 for HomeKit communications.  To allow the Web Interface to use port 80 instead, the HomeSpan port must be changed to something else using `homeSpan.setPortNum(port)`.  In this sketch the port was changed to 1201 (an arbitrary number).
  * HomeSpan normally reserves 8 TCP connections for HomeKit controllers.  Since WebServer consumes 2 TCP sockets (one for the server, and one for the client), we need to instruct HomeSpan not use use all 8 TCP connections for itself.  Additionally, this sketch enables OTA updates, which also needs a TCP socket.  The total number of TCP sockets reserved for HomeSpan must therefore be reduced to 5 using the method `homeSpan.setMaxConnections(5)`.
  * The WebServer cannot be launched until the device has established WiFi connectivity.  The code to setup the WebServer is wrapped in the function `setupWeb()` and launched via the callback `homeSpan.setWifiCallback(setupWeb)`.

---

### Feedback or Questions?

Please consider adding to the [Discussion Board](https://github.com/HomeSpan/HomeSpan/discussions), or email me directly at [homespan@icloud.com](mailto:homespan@icloud.com).




