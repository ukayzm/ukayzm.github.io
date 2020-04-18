---
title:  "Using Arduino CLI in command line interface"
tags:   arduino
feature-img: "assets/img/posts/face_clustering.png"
thumbnail:   "assets/img/posts/face_clustering.png"
date:   2020-04-17 12:00:00 +0900
layout: post
---

## Arduino CLI

Check [Arduino-CLI in GitHub](https://github.com/arduino/arduino-cli)

## Install

```bash
$ curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh

$ sudo usermod -a -G dialout $USER         # add this user to dialout group
                                           # You need to reopen the terminal or reboot the computer

$ id $USER                 # check that you have 20(dialout)
uid=1000(rostude) gid=1000(rostude) groups=1000(rostude),4(adm),20(dialout),24(cdrom),27(sudo),30(dip),46(plugdev),116(lpadmin),122(sambashare)

$ arduino-cli config init
Config file written to: /home/rostude/.arduino15/arduino-cli.yaml

$ arduino-cli core update-index
Updating index: package_index.json downloaded

```

## Config for cheaper board

```bash
$ dmesg
[ 3059.609249] usb 1-1: new full-speed USB device number 6 using xhci_hcd
[ 3059.759268] usb 1-1: New USB device found, idVendor=1a86, idProduct=7523
[ 3059.759284] usb 1-1: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[ 3059.759294] usb 1-1: Product: USB2.0-Serial
[ 3059.761263] sdhci-pci 0000:00:12.0: SDHCI controller found [8086:2296] (rev 35)
[ 3059.821491] usbcore: registered new interface driver usbserial_generic
[ 3059.822413] usbserial: USB Serial support registered for generic
[ 3059.826341] usbcore: registered new interface driver ch341
[ 3059.827087] usbserial: USB Serial support registered for ch341-uart
[ 3059.828153] ch341 1-1:1.0: ch341-uart converter detected
[ 3059.828944] usb 1-1: ch341-uart converter now attached to ttyUSB0
[ 3059.829169] sdhci-pci 0000:00:12.0: SDHCI controller found [8086:2296] (rev 35)
[ 3059.830108] sdhci-pci 0000:00:12.0: SDHCI controller found [8086:2296] (rev 35)
```


```bash
$ arduino-cli board list      # Check the connection to the board
Port         Type              Board Name              FQBN                 Core
/dev/ttyACM0 Serial Port (USB) Arduino Uno             arduino:avr:uno      arduino:avr
/dev/ttyACM1 Serial Port (USB) Arduino/Genuino MKR1000 arduino:samd:mkr1000 arduino:samd
/dev/ttyUSB0 Serial Port (USB) Unknown
...

$ arduino-cli core install arduino:avr      # Install Arduino:avr core
Downloading packages...
arduino:avr-gcc@7.3.0-atmel3.6.1-arduino5 downloaded
arduino:avrdude@6.3.0-arduino17 downloaded
arduino:arduinoOTA@1.3.0 downloaded
arduino:avr@1.8.2 downloaded
Installing arduino:avr-gcc@7.3.0-atmel3.6.1-arduino5...
arduino:avr-gcc@7.3.0-atmel3.6.1-arduino5 installed
Installing arduino:avrdude@6.3.0-arduino17...
arduino:avrdude@6.3.0-arduino17 installed
Installing arduino:arduinoOTA@1.3.0...
arduino:arduinoOTA@1.3.0 installed
Installing arduino:avr@1.8.2...
arduino:avr@1.8.2 installed

$ arduino-cli board listall                 # Confirm the installation
Board Name                          FQBN
Adafruit Circuit Playground         arduino:avr:circuitplay32u4cat
Arduino BT                          arduino:avr:bt
Arduino Duemilanove or Diecimila    arduino:avr:diecimila
Arduino Esplora                     arduino:avr:esplora
Arduino Ethernet                    arduino:avr:ethernet
Arduino Fio                         arduino:avr:fio
Arduino Gemma                       arduino:avr:gemma
Arduino Industrial 101              arduino:avr:chiwawa
Arduino Leonardo                    arduino:avr:leonardo
Arduino Leonardo ETH                arduino:avr:leonardoeth
Arduino Mega ADK                    arduino:avr:megaADK
Arduino Mega or Mega 2560           arduino:avr:mega
Arduino Micro                       arduino:avr:micro
Arduino Mini                        arduino:avr:mini
Arduino NG or older                 arduino:avr:atmegang
Arduino Nano                        arduino:avr:nano
Arduino Pro or Pro Mini             arduino:avr:pro
Arduino Robot Control               arduino:avr:robotControl
Arduino Robot Motor                 arduino:avr:robotMotor
Arduino Uno                         arduino:avr:uno
Arduino Uno WiFi                    arduino:avr:unowifi
Arduino Yún                         arduino:avr:yun
Arduino Yún Mini                    arduino:avr:yunmini
LilyPad Arduino                     arduino:avr:lilypad
LilyPad Arduino USB                 arduino:avr:LilyPadUSB
Linino One                          arduino:avr:one
```

## create sketch

```bash
$ mkdir sketch/
$ cat > sketch/sketch.ino        # create sketch.ino
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(115200);
  Serial.println("start...");
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("H");
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  Serial.println("L");
  delay(1000);
}
^D

$ ls -Al sketch/           # sketch.ino is created
total 4
-rw-r--r-- 1 rostude rostude 175 Apr 16 10:25 sketch.ino
```

### Compile and upload the sketch

```bash
$ arduino-cli compile --fqbn arduino:avr:uno sketch/          # Compile 'sketch' directory
$ arduino-cli compile --fqbn arduino:avr:nano:cpu=atmega328old sketch/          # Compile 'sketch' directory for nano with ATmega328P (Old Bootloader)
Sketch uses 924 bytes (2%) of program storage space. Maximum is 32256 bytes.
Global variables use 9 bytes (0%) of dynamic memory, leaving 2039 bytes for local variables. Maximum is 2048 bytes.

$ ls -Al sketch/
total 28
-rw-r--r-- 1 rostude rostude   175 Apr 16 10:25 sketch.ino
-rwxrwxr-x 1 rostude rostude 14112 Apr 17 16:11 sketch.ino.arduino.avr.uno.elf*
-rw-rw-r-- 1 rostude rostude  2615 Apr 17 16:11 sketch.ino.arduino.avr.uno.hex
-rw-r--r-- 1 rostude rostude  3976 Apr 17 16:11 sketch.ino.arduino.avr.uno.with_bootloader.hex

$ arduino-cli upload -p /dev/ttyACM0 --fqbn arduino:avr:uno sketch/    # upload the sketch
$ arduino-cli upload -p /dev/ttyUSB0 --fqbn arduino:avr:nano:cpu=atmega328old sketch/      # for nano with ATmega328P (Old Bootloader)
```

## Using serial output

```bash
$ stty -F /dev/ttyUSB0 cs8 115200 ignbrk -brkint -icrnl -imaxbel -opost -onlcr -isig -icanon -iexten -echo -echoe -echok -echoctl -echoke noflsh -ixon -crtscts
$ cat /dev/ttyUSB0 
start...
H
L
H
L
H
...
^C

$ echo "Hello Arduino" > /dev/ttyUSB0
$ echo -n "H" > /dev/ttyUSB0
```

## Add library

```bash
$ arduino-cli lib search debouncer     # search library
Name: "BirdhouseSDK"
    Author: Serhiy Korzun <korzun.serhiy@gmail.com>
    Maintainer: Prometheus team <prometheus.larp@gmail.com>
    Sentence: An Arduino library to easy control lots of relays effects, buttons with debouncer, analog indicators and more
    Paragraph: BirdhouseSDK uses a non-blocking approach and can control relays and analog indicators in simple (on/off) and complex (serial blinking, random blinking) ways in a time-driven manner.
    Website: https://github.com/Nargott/birdhouse_sdk
    Category: Other
    Architecture: *
    Types: Contributed
    Versions: [1.0.0]
Name: "Debouncer"
    Author: hideakitai
    Maintainer: hideakitai
    Sentence: Debounce library for Arduino
    Paragraph: Debounce library for Arduino
    Website: https://github.com/hideakitai
    Category: Timing
    Architecture: *
    Types: Contributed
    Versions: [0.1.0]
Name: "FTDebouncer"
    Author: Ubi de Feo
    Maintainer: Ubi de Feo, Sebastian Hunkeler
    Sentence: An efficient, low footprint, fast pin debouncing library for Arduino
    Paragraph: This pin state supervisor manages debouncing of buttons and handles transitions between LOW and HIGH state, calling a function and notifying your code of which pin has been activated or deactivated.
    Website: https://github.com/ubidefeo/FTDebouncer
    Category: Signal Input/Output
    Architecture: *
    Types: Contributed
    Versions: [1.3.0, 1.3.2]
...

$ arduino-cli lib install FTDebouncer     # install library
FTDebouncer depends on FTDebouncer@1.3.2
Downloading FTDebouncer@1.3.2...
FTDebouncer@1.3.2 downloaded
Installing FTDebouncer@1.3.2...
Installed FTDebouncer@1.3.2

```

## FQBN of the unknown board

The easiest way to determine the fqbn of a board is:
Start the Arduino IDE GUI (Yes, I know you are dead set on using the CLI only but it really won't hurt too badly to use the GUI every once in a while)
File > New
File > Preferences > Show verbose output during: > compilation (check) > OK
Select the board from the Tools > Board menu
If there are additional configurations of the board necessary, select those from the appropriate Tools menus
Sketch > Verify/compile

After the compilation finishes scroll up to the top of the black console window at the bottom of the Arduino IDE window. You will see a command that looks something like this:
```
C:\Program Files (x86)\Arduino\arduino-builder -dump-prefs -logger=machine -hardware C:\Program Files (x86)\Arduino\hardware -tools C:\Program Files (x86)\Arduino\tools-builder -tools C:\Program Files (x86)\Arduino\hardware\tools\avr -built-in-libraries C:\Program Files (x86)\Arduino\libraries -libraries D:\Users\kjeom\Documents\Arduino\libraries -fqbn=arduino:avr:nano:cpu=atmega328old -vid-pid=1A86_7523 -ide-version=10812 -build-path C:\Users\kjeom\AppData\Local\Temp\arduino_build_337148 -warnings=none -build-cache C:\Users\kjeom\AppData\Local\Temp\arduino_cache_981687 -prefs=build.warn_data_percentage=75 -prefs=runtime.tools.avr-gcc.path=C:\Program Files (x86)\Arduino\hardware\tools\avr -prefs=runtime.tools.avr-gcc-7.3.0-atmel3.6.1-arduino5.path=C:\Program Files (x86)\Arduino\hardware\tools\avr -prefs=runtime.tools.avrdude.path=C:\Program Files (x86)\Arduino\hardware\tools\avr -prefs=runtime.tools.avrdude-6.3.0-arduino17.path=C:\Program Files (x86)\Arduino\hardware\tools\avr -prefs=runtime.tools.arduinoOTA.path=C:\Program Files (x86)\Arduino\hardware\tools\avr -prefs=runtime.tools.arduinoOTA-1.3.0.path=C:\Program Files (x86)\Arduino\hardware\tools\avr -verbose D:\temp\sketch_apr17a\sketch_apr17a.ino
```

You will find the fqbn of the board you had selected somewhere in that command.

-fqbn=arduino:avr:nano:cpu=atmega328old


## Reference

* [Arduino-CLI in GitHub](https://github.com/arduino/arduino-cli)

