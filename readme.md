# BLE-LED
Control a cheap Bluetooth Low Energy LED Strip Controller

## Aim
The aim of this repo is write a small libary for [this](https://www.aliexpress.com/item/33024703436.html?spm=a2g0s.9042311.0.0.4cb24c4dsnlJEF) LED-Strip controller from Aliexpress.

## Reverse Engineering
The only documentation is [this](https://play.google.com/store/apps/details?id=shy.smartled) App. I connected an old android phone to my PC and used WireShark to listen to the Bluetooth communication. I noted the sent payload to misc/commands.txt. The handle `0x0008` is used.

The next step was to find the device *ELK-BLEDOM* with `sudo hcitool lescan` on my PC and noted the MAC address. Then I used `sudo gatttool -b <MAC> -I` to connect to the device. First I have to connect with `connect` then I can send commands using `char-write-cmd 0x0008 <CMD>`.

## Commands

Each command consists of 9 bytes. I could figure out:

* The first byte is always `0x7e`
* The second byte doesn't matter, so I always will use `0x00`
* The third byte is the operation code
* The forth to the eighth byte are parameteres for the operation. Each Operation uses a different count of parameteres. The bytes after the parameters don't change everything and so I always will use `0x00` for the unneded bytes
* The ninth byte is always `0xef`

### Operation Codes

#### **01: brightness**
`7e 00 01 <BRIGHTNESS> 00 00 00 00 ef`\
parameters:
* Brightness: 0-100 (0x00 - 0x64)


#### **02: effect speed**
`7e 00 02 <SPEED> 00 00 00 00 ef`\
parameters:
* Speed: 0-100 (0x00 - 0x64)


#### **03: effect**
`7e 00 03 <EFFECT> 03 00 00 00 ef`\
parameters:
* Effect: 0x80 - 0x9c
  * 0x80: static red
  * 0x81: static blue
  * 0x82: static green
  * 0x83: static cyan
  * 0x84: static yellow
  * 0x85: static purple
  * 0x86: static white
  * 0x87: 3 color jump
  * 0x88: 7 color jump
  * 0x89: 3 color fade
  * 0x8a: 7 color fade
  * 0x8b: red pulse
  * 0x8c: green pulse
  * 0x8d: blue pulse
  * 0x8e: yellow pulse
  * 0x8f: cyan pulse
  * 0x90: purple pulse
  * 0x91: white pulse
  * 0x92: red-green fade
  * 0x93: red-blue fade
  * 0x94: green-blue fade
  * 0x95: 7 color flash
  * 0x96: red flash
  * 0x97: green flash
  * 0x98: blue flash
  * 0x99: yellow flash
  * 0x9a: cyan flash
  * 0x9b: purple flash
  * 0x9c: white flash
* Unknown: this parameter has to be `0x03`

#### **04: on/off**
`7e 00 04 <STATE> 00 00 00 00 ef`\
parameters:
* State:
  * 0xf0: ON
  * 0x00: OFF

#### **05 01: color black/white**
`7e 00 05 01 <BW> 00 00 00 ef`\
parameters:
* BW: 0-100 (0x00 - 0x64)
  * 0x00: Black (off)
  * 0x64: White (max) 

#### **05 02: temperature**
`73 00 05 02 <TEMP> 00 00 00 ef`\
parameters:
* TEMP: 0-100 (0x00 - 0x64)
  * 0x00: cold
  * 0x64: warm

#### **05 03: rgb**
`73 00 05 03 <R> <G> <B> 00 ef`\
parameters:
* R: 0-255 (0x00 - 0xff)
* G: 0-255 (0x00 - 0xff)
* B: 0-255 (0x00 - 0xff)
