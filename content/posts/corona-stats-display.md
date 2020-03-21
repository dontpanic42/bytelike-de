+++
title = "Building a Wireless Corona Statistics Display"
date = 2020-03-21T08:32:58.029Z
author = "Daniel"
cover = ""
tags = []
keywords = ["Corona", "COVID-19", "esp8266"]
description = "A look at how to build a simple, wireless corona statistics display based on an esp 8266"
showFullContent = false
+++

Since the outbreak of the corona virus, my weekends have gotten (even more) boring. While looking at google news for the 20th time in a row, I suddenly got the idea that it would be pretty neat to have a live display showing the latests corona statistics for my country. Looking through my random junk drawer, I realized that I've got everything I need at home! With nothing better to do, I went to work...

Since I'm not a big fan of lots of cables in my living room, I decided to use an old esp 8266 I had lying around as the basis for the build.

![esp8266](/img/posts/corona-stats-display/esp8266.jpg)

The esp 8266 really is a beast of a microcontroller. Boasting an 80 mhz 32-bit microprocessor, (up to) 4 MB flash and more than 100kb ram, it's much more powerfull then my normal go-to board, the arduino nano. But more importantly it features build in WiFi capabilities! It's really the ideal setup to play around with IoT stuff. And did I mention that the development boards (I use the NodeMCU v3) are also dirt cheap? You will usually get 3 of them for 12 Euros on Amazon, and you will probably get them even cheaper if you are prepared to check ebay or wait for them to be delivered from china.

![display](/img/posts/corona-stats-display/display.jpg)

The second important component of the build is the display. Here I again grabbed whatever I found in my drawer. I had a generic 128x64 two color OLED hat hand. You can find them all over Amazon, or you can search for the chipset, "SSD 1306". The great thing about these displays is that you can use an Adafruit Libarary ([Base Library](https://github.com/adafruit/Adafruit_SSD1306), [Fonts and Graphics Library](https://github.com/adafruit/Adafruit-GFX-Library)) that is well document and easy to use. Searching for "Adafruit SS3 1306" on google will also result in a ton of tutorials and learing resources. And again - the displays are pretty cheap. About 13 Euros for three displays on Amazon, probably much cheaper elsewhere.

The way to connect the esp 8266 to the display varies based on the type of development board and the type of display you use. For my setup (NodeMCU v3 and generic SSD 1306) looked like this:

| NodeMCU v3 Pin | SSD 1306 Pin | 
|:--------------:|:------------:|
| 3V             | VCC          |
| GND            | GND          |
| D1             | SCL          |
| D2             | SDA          |

When you search google for "NodeMCU v3 SSD 1306" you will again find a ton of tutorials and videos explaining how to wire the components up.

Assembly was pretty straight forward. Since I still hope this will blow over eventually, I did not want to spend too much time making this device presentabel. I mean - either this will be obsolete in a few weeks or month, or it really is the appocalypse, and in that case I don't think build (and code) quality doesn't really matter that much :-)

I dediced to build the device as a stack, with the display mounted on a mounting plate and the NodeMCU at the bottom, connected with lots of brass stand-offs. The mounting plate was designed in Fusion360 and printed out in about 40 minutes on my trusty Crealty Ender 3. I added a big red button below the display a) because I think a device like this needs a big red button and b) just in case want to extend the firmware in the future. But for now it's not connected. 

![The finished device](/img/posts/corona-stats-display/device.jpg)

It's pretty hard to get the display show up correctly on a photo - it seems like there is some kind of rolling shutter effect going on. In reality the display looks really nice and crisp.

With the hardware out of the way, it was time for coding. And again this was straight forward. The esp 8266 package for the Arduino IDE features some really neat libraries, like one for WiFi that lets you connect to your local WiFi with just a few lines of code, a network client lib that deals with tls encryption and a simple https library that lets you send and receive http requests. 

As a data source, I used [this api](https://github.com/novelcovid/api), which gives you (near) real time data for your country. The api is really easy to use, just query [https://corona.lmao.ninja/countries/germany](https://corona.lmao.ninja/countries/germany) (replace germany with your country) and it will return the number of cases, number of active cases, deaths and so on. For now I'm only displaying total, active and new cases, but it would be trivial to add e.g. a trend display, graphs and so on.

That's all for now. As always, you can find my source code (sorry, it's not very nice or optimized...) and the 3D files for the mounting plate on github: [https://github.com/dontpanic42/esp8266-corona-display](https://github.com/dontpanic42/esp8266-corona-display)

Thanks for reading and stay healthy!