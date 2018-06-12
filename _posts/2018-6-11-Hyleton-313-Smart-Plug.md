---
layout: post
title: Changing the firmware on Hyleton 313 Smart Plug
---


## Open the case
Opening the case didn't look as easy as it was with other (i.e. Sonoff) devices, because there were no visible screws. However it was not that difficult using some sharp plastic pry tool and a heat gun (hair dryer on max setting could probably also work, but I had my heat gun ready). The pry tools I used look like these:
![pry tools](../images/hyleton-313/pry_tool.jpeg "pry tools") But anything similar will do the trick, just work your way slowly, pry from the middle of each side. Then slowly slide the tool towards the corners and the case will ease enough eventually - open it with a gentle pull. 

Once the case is open, the top side of the PCB will be exposed:

![top-side](../images/hyleton-313/pcb-top.jpg "PCB top side")

The WiFi module is soldered vertically to the main PCB and sits right next to the relay. In order to get access to its pins, I had to first remove the screw from the centre of the PCB. After that the bottom plastic plate, which holds the three mains connector prongs, could be moved a bit to the side without desoldering anything (it is attached with short cables to the PCB, but cables' length was just enough to move it out of the way of the WiFi module's pins).

Here's a view of the bottom side of the PCB:

![bottom-side](../images/hyleton-313/pcb-bottom.jpg "PCB bottom side")

You can see the product labels (product code, date and board revision), as well as the UL number. But what I was really interested in were the pins of the WiFi module.

## Identify WiFi module
This was the first time I encountered this kind of WiFi module. It looked quite similar to the one I had already seen in another smart plug: [TYWE2S](https://docs.tuya.com/en/hardware/WiFi-module/wifi-e2s-module.html). However, the new module didn't have any labels on its bottom side (facing away from the relay). Unfortunately I couldn't glimpse what (if anything) was written on the front side of the module, because it was soldered quite close to the relay. So having no clue what kind of module I had, I started tracing the power pins - the 3.3V output from the LDO regulator and GND. I quickly found that the two rightmost pins of the WiFi module were in fact the power pins. I applied 3.3V and GND to those pins from an external power supply and bingo - the blue LED started to blink! I even registered the module in the "*Smart Life*" App while it was powered through the low voltage power supply.
Then I had to find the serial (*Tx*/*Rx*) pins and of course *GPIO0*. Finding *Tx* pin was easy - it occurred to be the pin next to the GND pin. When I applied power to the WiFi module, the following debug messages appeared on the terminal (baud rate set at 74880):

```
ets Jan  8 2013,rst cause:1, boot mode:(3,7)

load 0x40100000, len 1396, room 16
tail 4
chksum 0x89
load 0x3ffe8000, len 776, room 4
tail 4
chksum 0xe8
load 0x3ffe8308, len 540, room 4
tail 8
chksum 0xc0
csum 0xc0

2nd boot version : 1.4(b1)
  SPI Speed      : 40MHz
  SPI Mode       : QIO
  SPI Flash Size & Map: 8Mbit(512KB+512KB)
jump to run user1 @ 1000

OS SDK ver: 1.4.2(23fbe10) compiled @ Sep 22 2016 13:09:03
phy v[notice]device.c:784 fireware info name:esp_yuan_plug_America version:1.0.0
[notice]device.c:785 fireware PID=<MyPID>
[notice]device.c:786 fireware for HAI-LONG-TONG
[notice]gw_intf.c:333 Authorization success
[notice]device.c:816 ##  AUTH DONE ##

[notice]key.c:77 rst reason:0

[notice]key.c:77 rst reason:0

mode : sta(dc:4f:22:xx:xx:xx)
add if0
scandone
state: 0 -> 2 (b0)
state: 2 -> 3 (0)
state: 3 -> 5 (10)
add 0
aid 3
pm open phy_2,type:2 0 0
cnt

connected with <MyApName>, channel 4
dhcp client start...
ip:192.168.100.220,mask:255.255.255.0,gw:192.168.100.1
[notice]mqtt_client.c:610 mqtt connect success

```

*PID*, *MAC* and my *AP* name are censored above, everything else is left as it was.

Some interesting findings from the messages above:

- The firmware (or *fireware* as it is incorrectly spelled) seems to be made for a company called [HAI-LONG-TONG](http://usb-wall-charger.sell.everychina.com/aboutus.html), which owns the brand HYLETON (these two sound quite similar!)
- The SPI flash chip size is 8Mbit (1MB)

Surely no information about pins assignment could be seen in the debug messages, so it was time to move onto identifying those pins.

## Identify pins
Having already identified three of the most important pins (Vcc, GND and Tx), I thought that my task was almost over. I just needed to find the positions of Rx and GPIO0. But my curiosity was strong at that moment and I started to look for the control signals first. Here is a close-up view of the module's pins as seen from the bottom of the main PCB:
![pcb-bottom-module-pins](../images/hyleton-313/pcb-bottom-module-pins.jpg "module pins")
As seen in the picture above, pins 1,2,3 and 12 have no corresponding pads on the main PCB (i.e. they are not connected). Probing the other pins I found the connection to the button - when pressed, it connects GND to pin 5. This in turn switched the relay on, as well as the red LED. Probing around I found that when the relay was switched on, the signal on pin 6 went low (it stayed high when the relay was switched off). Soon I also found that the red LED was switched on when the signal on pin 7 went low, and blue LED was switched on when the signal on pin 8 went low. Using the pin numbering scheme above, *Vcc* was on pin 13, *GND* - on pin 14, and *Tx* - on pin 12. So my assumption was that *Rx* and *GPIO0* could be routed to some of the remaining pins - 4, 9, 10 or 11. I also assumed that *Rx* pin could be routed in close proximity to the *Tx* pin (on other similar modules *Rx* and *Tx* are usually routed next to each other). So I just soldered a jumper wire to pin 10, hoping that it was indeed the *Rx* pin I was looking for. Then I needed to just connect the pin corresponding to *GPIO0* (unknown to me at the time) to ground while applying power to the module, in order to bring its bootloader into UART firmware upload mode. Unfortunately this exercise didn't work out as planned, so I resorted to the desoldering braid. After removing all the solder from the module's pads, I had just pulled the module out of the slot of the main PCB. I hoped to find some identification at the front, so I can lookup the pinout:
![module-front](../images/hyleton-313/module-front.jpg "module front")
Nothing! Only components' labels, but no manufacturer or model identification whatsoever. Here is a photo of the back of the module as well, just for completeness:
![module-back](../images/hyleton-313/module-back.jpg "module back")


## Dump original firmware

## Upload custom firmware

## Configuration
