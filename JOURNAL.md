# Twisted? What's That?

In early April, 2026, I found about [this tiny IPS LCD](https://www.buydisplay.com/0-42-inch-mini-color-tft-lcd-display-module-96x54-ips-st7735) panel from AliExpress, it measured only 0.42" and I felt like I should make something using it.

![picture showing one of those tiny 0.42" displays doing action](https://cdn.hackclub.com/019fb916-9cae-7376-8436-6bdd73e8b96f/output.webp)

So I decided to build a tiny development board around it, nothing really came in mind best at the time other than the ESP32-S3, as it had WiFi, BLE, USB and two quite powerful Xtensa LX7 cores with vector insturctions. I felt like it could serve as a powerful base that would be easily extended with other "shields" later down the line.

So I designed a tiny board around the ESP32-S3FN4R2 which has both the processor and extra memory in package, of course alongside that tiny display.

![picture highlights PCB outline as well as the various layers](https://cdn.hackclub.com/019fb8c6-3ad9-7d13-a7d6-e995db1b5055/image.png)

The board I designed at the time was made around parts I was able of finding locally, so I was quite limited by my parts selection.

At the end I finished the board and called it "Foton" as a code name, thought nothing of it and shared it online to the organisation's [Instagram profile.](https://www.instagram.com/p/DWvD35siBwf)

And then... nothing. It was left to dust, similar to a bunch of other projects, the reason behind leaving it behind was because of lack of funding for side projects.

Before I quit the project I had started working on a DAC/ADC shield for the thing based on the ES8388 codec, it was never completed as I had already moved onto other, slightly more yielding projects.
![](https://cdn.hackclub.com/01a07244-3aa9-7892-98d8-664770dea844/image.png)
![](https://cdn.hackclub.com/01a07243-ca59-7ad0-94ad-13e85be1a8bc/image.png)

It was a headache, just thinking about a budget for such a project, given that customs are calculated randomly here in Egypt and often overshot due to ordering the minimum amount from JLCPCB, which is 5 boards, it makes them think that we are some kind of a reseller.

Foton sat for a good while like this, until early August when I decided to revisit the project and maybe shine new a beam onto it.

When I opened the design files, I realized the sad state of the schematic, turned out I had a different mindset four months back. I also had made some questionable routing decisions.

![picture showing the current state of the schematic](https://cdn.hackclub.com/019fb8cb-9db4-7e82-8c9d-6091fba70284/image.png)

I also realized that due to lack of space, which was cased by the size of the components I had used (limited by the local market), I decided to ditch battery management off board and instead leave it to an external shield.

Anyways, I decided to do a few things to Foton, so here is the plan.

- Foton becomes "Twisted". We decided to give it a rebrand under our open-source hardware organization, [Triax-Labs](https://github.com/Triax-Labs).
- Twisted should be a radically different design built around the outlines that the original prototype provided.
- Bring back the missing important traits like battery management (maybe with USB-OTG) and a wireless antenna that is built-in.
- Introduce more add-on/shield boards.

The idea behind Twisted is that it should be completely expandable using its bottom row of pin headers (02x10 pin, 1.27mm pitch.)

We are taking lots of inspiration from M5Stack's boards.

_We also need to find a small LiPo battery to use alongside the finished product._

---

# Starting from a Blank Slate

First, before doing anything we must get rid of all the existing connections and components as we are going to select everything again later.

![](https://cdn.hackclub.com/019fc368-c1cf-789d-863a-a01e8a05f527/image.png)
![](https://cdn.hackclub.com/019fc36b-7492-76e3-840c-bd6cd619dbd3/image.png)

All components that are going to change were removed, the schematic was cleaned up too.

I also have picked an antenna (`KH-3216-A27`)! It took a good while of research and lots of scrolling in `fcc.id`.

The ESP32-S3FH4R2 was kept in place, alongside its crystal. Connectors were also left as I'm using the exact outline as the original prototype.

## Picking a Battery Charger

Now, before I start wiring anything, a battery charger circuit must be put in place as the entire schematic should be put around it.

In the past I had made a long list of tiny charging ICs that I could later use.

![](https://cdn.hackclub.com/01a07c8e-aac1-74e6-9f8e-a2a6aa163676/image.png)
![](https://cdn.hackclub.com/01a07c8f-3646-7bbd-9b66-97868d6fb39e/image.png)

However, this list didn't include a charger that had the requirements of my exact use case.

These requirements are: **NVDC power path**, **USB-OTG** boosting and a documented **I2C interface** as long as a package that is both tiny and **economically assembled at JLCPCB**.

After a good day of searching on JLCPCB's parts list, I came out with two candidates, `BQ2562xR` and the Chinese `BCT24157`.

Now the `BCT24157` comes at a really cheap price even for low quantities, however it comes in a BGA package which might be a bit expensive to assemble.
On the other hand the TI part `BQ2562xR` comes in a weird QFN-based package, but at an _expensive_ retail price per unit.

![](https://cdn.hackclub.com/019fc423-0f49-7410-baca-82f58d5b2515/image.png)

Choosing the `BCT24157` is quite risky, as its documentation quite lacks a lot of crucial information, as well as being a BGA package it could've ended in paying much more in PCBA.

I ended up picking the `BQ25628R`, as it comes with a less aggressive feature set than its sibling the `BQ25620R` which means a lower price, I wasn't going to use those features anyways.

## Boost Inductor Woes

A feature I always wanted to see on other platforms is USB-OTG, it allows people to use external peripherals when the device is being powered completely from battery, so users could use flash drives, keyboards and mice easily.

One issue though, USB-OTG requires a boost convert that in term needs a power inductor.

Usually power inductors are big and bulky, specifically for my application where every literal millimeter counts.

I'm was pretty certain that the isles of LCSC had something waiting for me, so I spent about 2 hours looking for an inductor that matches the requirements of the charger IC.

Eventually I found this inductor: `APH160808C1R0MX02` it is tiny (0603), has enough oomph and seems perfect for this exact application (which to be fair, did sound a bit suspicious to me at first.)

![](https://cdn.hackclub.com/01a07da8-4db7-774f-9384-8b0ded261802/image.png)

Now it all takes shape!

## Keypad and GPIO Expander

Twisted has a tiny set of 7 buttons on its front, serving as a keypad, 6 of those buttons are mapped for navigation and input, and the remaining one is connected to the BOOT pin of the MCU (which means it can be also used for input after the MCU boots up.)

The issue is, wiring 6 buttons to the MCU directly is quite wasteful and will result in less GPIOs exposed to the expansion header, that's why a GPIO expander was needed.

Finding a tiny GPIO expander that is cheap and has more than 4 GPIO slots was quite hard, I eventually found the `FXL6408UMX` which comes in a `1.8x2.6mm` QFN-like package.

![](https://cdn.hackclub.com/01a07dae-129c-768a-ba77-175e6058b5a2/image.png)

## Voltage Regulation

The battery charger puts out `VBAT+` (3.3V - 4.2V) or `VBUS` (5V) on pin `VSYS` based on the charger presence. Thus, voltage regulation is a critical piece of the puzzle, the ESP32-S3 needs 3.3V in order to function, supplying `VSYS` directly would fry the hell out of it.

This one was easy though, as I had already used this part `LW5233N11E` many times before, it proves to be perfect for this application.

It is quite tiny measuring only `1x1mm`, as well as 500mA of output current (that's a lot compared to the size of the package.)

![](https://cdn.hackclub.com/01a07dc3-6c83-7005-a350-825b665f3c7e/jlcpcb1_2_.webp)
_Image of a different hardware project, the highlighted circles are the `LW5233N11E` voltage regulator in action._

## Meet Twisted!

Now the schematic is about complete:
![](https://cdn.hackclub.com/01a07dc5-f049-720f-b53d-c473d835f54e/image.png)

And so is the board (skipping lots of routing pain.)

![](https://cdn.hackclub.com/01a07dcd-f21f-7968-8a00-245b88ea2e28/twisted-rendered-back.png)
![](https://cdn.hackclub.com/01a07dcd-edd5-7d68-ab97-722aa4a15e99/twisted-rendered-front.png)

Say hi to Twisted :D

---

# Twisted Listen

Remember I've talked about an audio addon for Foton before it becomes Twisted?

I've revised that too, and it's now better than ever!

![](https://cdn.hackclub.com/01a07dd4-6ed8-747a-877e-fb85dad85dc5/twisted-listen-rendered-front.png)
![](https://cdn.hackclub.com/01a07dd4-6b17-7878-9f14-fcaad158ace0/twisted-listen-rendered-back.png)

It is now based on the `ES8316` audio codec, which offers two ADC channels and two stereo DACs which allowed me to put a microphone on board as well as a small audio amplifier that could be used for adding a loudspeaker for Twisted.

---

# Twisted Connect

I have uploaded a new shield, Twisted Connect on October the 8th (a11f0e1c3315e99503966e0c169e91b8d9091f87)...

![](https://cdn.hackclub.com/01a1195e-e720-7408-bbb9-da0d3babfab9/image.png)
![](https://cdn.hackclub.com/01a11960-f23b-7eb3-b9da-d02fe3e75747/image.png)
![](https://cdn.hackclub.com/01a11968-6706-772b-8d75-7074c0f8adf8/twisted-connect-layer-overview.gif)


Twisted Connect is a shield for the Twisted Bus that provides the functionality of an extra microcontroller, it adds on a CH585 MCU which comes with an interesting set of peripherals, NFC/RFID (with support for reading, writing and emulating tags,) both a USB High Speed and a USB Full Speed controller as well as a BLE / 2.4GHz modem.

This shield could be used for either cooperative MCU communications (using best features from both boards) or it could be used to deep sleep the main MCU (the ESP32-S3) on the Twisted to preserve power.