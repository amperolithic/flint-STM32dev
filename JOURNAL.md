---
title: "Flint Devboard"
author: "Reuben 0"
description: "Imitating flint, this zero-power custom STM32 devboard slowly builds up potential and strikes a quick wireless spark to transmit Bluetooth or Wi-Fi packets." 
created_at: "2026-11-15"
---
# November 15: Initial Research + Schematic Startup

I have been wanting to try and make a devboard for a while. I think it would be a good introductory project to either PCBA or soldering with a hot plate (we shall see what option I choose). 

I originally thought of following the RP2040 devboard guide on Forge, but if I did that I would just be making a devboard for the sake of it. I wanted to create something with a somewhat unique feature. So I started looking for some more “unique” devboard ideas online and stumbled on a [devboard guide](https://www.instructables.com/Custom-STM32-Boards-From-Scratch-the-Complete-Guid/) for the STM32. As I was reading through it, I learned that some STM32 boards could manage power more efficiently than the ESP32, a microcontroller I considered using for a wireless macropad but decided against it because of its high power consumption (leading to a low battery life). This inspired me to make an STM32 devboard that had equivalent functionality to an ESP32 but used much less power. To make this more unique and emphasis my board’s low power functionality, I decided I am going to make the dev board a “Zero-idle energy harvester” by adding an energy-harvesting power management IC (PMIC) and a load switch so my board becomes a maintenance free platform (can be run on tiny solar cells or thermoelectric generators). 

With my plan now made, I decided I would follow the guide I mentioned earlier while adding in my own modifications. I also found a guide on [Fabricate](https://fabricate.hackclub.com/docs/guides-devboard#doc) that I might also reference as I am making this. I made a brief list of the things I will need to change or add to my project and began to start the PCB schematic with the help of Google and also some help from its AI search overview. At this point, I finally remembered to record my progress using Laspe. During the lapse I created a new KiCAD project, added the schematic for the STM32U575RIT6 and attempted to start wiring the power system while referencing its datasheet.

I chose the STM32U575RIT6 as the STM32U5 is a high performance ultra-low-power STM32. I learned the `R` indicates it is a 64 pin package, which is the perfect amount of mins I need. Any fewer, such as the 48 pin chip package, would leave too few free GPIO pins after I added stuff like the wifi module, USB, debugging and crystals, whereas anything more would be too wide for the compact stamp style board I am aiming to make. The `I` variant maximizes the Flash and SRAM to ensure the wireless module does not run out of memory, the `T` (LQFP package) is hand solderable. It was also one of the first chips I found during my search that was available. (The 6 just stands for the industrial temperature range, which I believe is standard)

I went into this project thinking I was going to use SMPS for power instead of LDO to further increase efficiency and lower power intake as I discovered the LDO version draws roughly twice as much current while the CPU is actively crunching numbers at 160 MHz. However asI was reading the datasheet I noticed my schematic was missing some pins for SMPS. I then realized I had incorrectly chosen to use the STM32U575RIT6 instead of the STM32U575RIT6Q, which had SMPS capabilities. I considered switching but I found that the STM32U575RIT6Q was really hard to find in stock. Although there was a STM32U575CIT6Q, I really wanted to keep the 64 pin package, so I decided to stick with using LDO, which was more readily accessible. Furthermore, its deep sleep current (which it will spend most of its time in) is basically the same as the SMPS, its routing is simpler and it gives me 4 more GPIO pins to work with. 

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/9Q6iECUoty0G)

I added two hours of time to account for the time I spent researching prior to recroding the lapse.

**Total time spent: 2.65 hours**

# November 16: Schematic Work - Power Supply and LSE

Did a work session today where I finished up the Power Supply wiring I started yesterday. Before I began I did some searching on Google regarding how to read the datasheet properly, and so I was no longer confused on what capacitors I needed to place. I think what got me so confused yesterday was that fact I was too focused on trying to match the power supply scheme to the one in my guides, when they had notably different designs. I now understand that when the datasheet reads "n x 100 nF + 1 x 10 μF", for instance, it is saying that a `n` amount of 100nF ceramic capacitors (that being each instance of the associated pin - VDD, in this case - as not every STM32U575xx microcontroller has the same amount of pins) should be placed so each pin has its own capacitor. In addtion, a 10μF bulk capacitor should be placed, and this capacitor is shared among all the VDD pins. It was also when I was doing this that I learned it is standard to add a 100nF capacitor to VBAT (even if the power scheme did not show it) if I am not adding a seperate coin cell to my board, and that adding a ferrite bead to VDAA can create cleaner ADC sensor reading (I actually discovered this when as I was making the schematic when I was checking over my design to make sure it was electrically correct).

When I was doing research yesterday, I learned that I don't need to add a 8MHz crystal (contrary to the guides I was following) as the U5 series, being newer, has a sufficent internal oscillator designed to be used with the USB. However, I did still had to add Low Speed External clock for low power operation (which is the main focus of my devboard.) So after finishing my power scheme I decided I would try and add that too. Honeslty, it took more time finding hwat crystal I wanted and getting its schematic into KiCad than actually wiring it. The wiring was pretty simple as I had both the datasheet and guides to reference. For my load capacitance calculation (C = 2(CL - Cparasitic)) I did `2*(7pF - 3pF)`.

I did spend a bit more time than expected on making the schematic look nice, but I really want to make sure my schematic looks organized. I used Fabricate's images as a starting point and then continued adjusting for clarity by myself. I know communication is important, and my inital wiring (which I used to get my bearing on which compnents went where) was a lot harder to read upon first glance. 

Tommorow I will either continue RoboticWorx guide's flow and starting breaking out the MC's GPIOs or I will hook up the USB, SWD debugging and maybe my energy harvesting system.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/xZ3T5_3_5pFG)

**Total time spent: 1.27 hours**

# November 17: Schematic Work - SWD Headers

Today was a very quick session; I was quite busy and did not get a chance to work until it was quite late. Earlier today I decided I would add all components that used the microcontroller GPIO pins (Wifi/Bluetooth module, BOOST/RST buttons and SWD headers) before choosing what GPIOs I would break out. Then I would work on power stuff (the energy harvesting system, USB, LEDs, etc). So I began by getting the symbol and footprint for the ST67W611M1, the Wi-Fi 6 / BLE 5.4 module I chose earlier to add wireless capabilities to my devboard. As I was doing this I discovered there were 3 versions of the coprocessor module: a `-B` version with an embedded PCB antenna, a `-U` version with an external RF connector (I think its for projects in faraday cages) and a `-P` version that has a RF pin output. I chose the `-B` version as I don't need an external or custom antenna. It appeared to be the simplest and most compact option. 

I knew I would have to do some datasheet reading and additional research to find where the pins go (all I know right now is I need to connect the SPI bus), so I decided to work on that for tomorrow. I’ll aim to finish the other GPIO components tomorrow.

With the limited time I had today, following the Instructable guide’s suggestion, I added pin headers so one can use an external programmer designed specifically for Cortex-M microcontrollers to Serial Wire Debug (SWD). According to my understanding, it uses some of the STM32’s serial wire pins to communicate with an external debugger tool (e.g. the ST-LINK/V2), which has useful debugging functions such as being able to flash code, pause execution on any line of code and inspect live variables. The NRST and optional SWO (Serial Wire Output) pin, lets the external debugger reset the chip and receive high speed internal data that is printed to the terminal, respectively. I have been trying my best to follow [good schematic practices](https://codex.hackclub.com/user-contrib-guides/schematic-practices/) using HackClub’s guide, but I think I’m overthinking things a little bit. I spent too long contemplating on whether I should use global labels or local labels (when Laspe was paused). I think I’m going to focus more on getting my connections done properly first (for at least the direct connections to my microcontroller) before I go in and polish for clarity. I think this might speed up my schematic workflow. One last thing to mention, directed to my future self, is that I plan to solder pin headers vertically to be PCB during assembly.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/nKGTNsDK4SVg)

**Total time spent: 0.62 hours**

# November 18: Schematic Work - BOOT/RST Buttons

Today I tried to lock in for an hour, but honestly ended up feeling locked out, here’s what happened: I continued with my plan to finish components that required GPIOs. I started off strong, using the guidance of the Fabricate tutorial and the STM32U5 datasheet to hookup the flashing buttons (BOOT and RESET; I already had chosen what button module I would use on the 15th) to the microcontroller. The only hiccup I experienced was that the two guides had different ways of hooking them up. The Fabricate guide omitted adding an external pull-up to the NRST pin (for the RESET button) because the STM32 it was using already had an internal one whereas the Instructables guide still added one. It also added the recommended 0.1μf capacitor for the NRST pin (according to datasheet) to the BOOT pin, explaining its good to prevent *button bouncing* (causes false triggers upon pressing the button) from occurring. I initially decided to keep the extra components but on further thought, I am probably going to remove the extra pull-up resistor tomorrow. The main benefit of adding the resistor in my case (that I searched up when I came across this discrepancy) would be if my board was noisy - which it won’t be due to its small size. In fact, I probably will lose more keeping this as the datasheet said the internal resistor deactivates during internal resets, saving power. Adding that external resistor, which is always on, will make that power saving feature -  a key principle of this project’s design - useless.

Anyway, after that I *attempted* to start wiring my wifi module (the ST67W611M1) to my microcontroller, but quickly became a bit lost as I had no guide to refer to for this part. I had only previously researched the fact I wanted to use this module, not how it was wired, so I spent a good 20 minutes just trying to get my bearings. I first looked at the datasheet, but it was not much of a help. What really was useful was the Hardware Setup Wiki for the part, which specified what pins I needed to connect to my microcontroller. However, due to my inexperience with hardware, I got a bit confused at this part. I decided I would start with connecting the SPI bus, as I knew for prior reading I would need to connect the “slave” SPI pins to the corresponding “master” pins - I just needed to identify what pins they were on the MC. I took a bit of time to find the pin definition chart on the MC datasheet, but once I did I could not figure out where I had to connect `SPI_CLK` to. No pin definition described “CLK”, so I spent quite a bit of time trying to understand where it needs to go. It was only after searching up how SPI bus connections work that I saw “CLK” can also be referred to as “SCK” (among other things). With that I was able to finally figure out what pin it goes to and I was able to piece where the other SPI pins connect. The same thing occurred with addressing the `SPI_CS`, but I knew what I had to do and found it connected to “NSS” as well. 

However, I **could not** figure out where `CHIP_EN` went! I originally wanted to try and wire the module myself before consulting external help, but I think I fundamentally misunderstand how I am meant to connect this module to the microcontroller. So I decided to stop there and I plan to do some reading so I am ready for action tomorrow! My goal for tomorrow is to successfully hook up the wifi module and break out some GPIOs.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/44AadsadqH9w)

**Total time spent: 1.25 hours**

# November 19: Schematic Work - ST67W611M1

Okay, so I did a bit of reading before I worked today and I found out there are essentially two types of pins when connecting a slave component to a master MCU: fixed hardware buses and flexible control GPIOs. According to my understanding, the fixed hardware pins must go to  specific hardware pins on the MCU, and these are usually pins that carry continuous data or a clock waveform, such as the dedicated SPI bus or the UART pins (which needs specific pins for flashing firmware). So it was correct for me to look at the “Alternate Functions” column in the pin definition chart to see where to connect the wifi modules SPI to. However, the flexible pins, which usually just turn something on or off, select a chip, or trigger an alert, can be assigned to **any** available GPIO pins. That’s why I could not find a pin definition for the CHIP_EN as it did not need to connect to a specific pin. It is simply an enable output, which can turn on or off the module by setting a logic to HIGH or LOW.

Now that I had a better understanding of what I was doing, I was able to complete wiring the rest of the wifi module. I’ll definitely make sure to get it sanity checked to ensure I did everything correctly.
To finish off the wifi module wiring, I added a load switch to the wifi’s power rail. When I was doing research on making a “zero-idle” board during the 15th, I found a load switch was needed to physically disconnect the 3V3 power from the ST67W611M1 (by setting the load switch’s GPIO pin to low) after the MCU finished sending data over Wifi, dropping the Wifi module’s power consumption to 0.00 µA.

I’m currently trying to work for at least an hour everyday on this project (until it’s done), so since I still had some time left and thought no other components I would be adding required GPIO pins from the main MCU, I began looking looking for what GPIO pins supplied some functions I want my devboard to have. I was specifically looking for I2C pins, SPI and UART/USART pins as I know they are useful to breakout and can be used in a variety of projects. I also added some pin connectors symbols as I plan to break out 32 pins. By the time I found those my hour had come to the end and I was also unsure of what other pins I wanted to breakout. I also realized in the process that I still needed to connect the USB (which requires GPIO pins from the MCU to send data). I plan on doing that tomorrow, along with polishing everything I have done.

Speaking of polishing, after finishing that last session, I tried to see if they were any way I could verify that I had wired my controller correctly (I made a few guesses), and that’s when I found out (from advice online) that a good way to check the wiring for a component is to check the manufacturer's reference schematic (of an official devboard that uses the chip you are using) and copy/reference the values on it. Little did I know I already had the necessary document open all of last session (I even looked at it!!) but I barely used it at all because I did not initially understand it. In the first session I opened it, I was looking at a completely wrong page that showed the other components. Then today, I was looking at it and was confused as to why so many parts were omitted (I now know it because manufacturers like to keep unpopulated footprints that can be temporarily populated later for potential testing), and had incorrectly came to the conclusion the pin assignments did not match my own schematic, so I should be weary of using it. So since I still had time before bed I decided to go over the wiring for the wifi module and I am now 99% sure it will work (will still sanity check it later though with input from others).

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/wP_p2aQVCzd9)

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/uDLbSnCLwqs5)

**Total time spent: 1.67 hours**

# November 22: Schematic Work - Polishing

I initially wanted to start working on connecting the USB receptacle, but once I found out which pins would be used by the USB (PA11 and PA12) I decided to clean up my schematic first, making sure to follow the best practices for schematic design. As I was doing that, I considered assigning the rest of my MCU breakouts, but then I remembered my "energy-harvesting" part will also require some GPIO connections, meaning I am going to have to work on my energy harvesting module (once I finishing connecting the USB) before I break out all my pins. With that revelation, I decided to get to work finding the exact components I would be using for my USB-C component. Once I decided on an USB and ESD protection (following a recommendation from the guide), I added them to my BOM and attempted to get the symbol, footprint and models for them, but for some reason my wifi or DigiKey was acting weird and downloads kept falling so it took a while to get them.

I will continue with the USB tomorrow.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/fA89dKHPzaw7)

**Total time spent: 1.05 hours**

# November 23: Schematic Work - USB-C Module

I successfully added USB-C to my schematic! I thought this was going to be a quick process as both guides showed examples of how my USB should look, however I was slightly thrown off by how my footprints were slightly different from the guides. I kept looking at the datasheet for specifically my USB-C to ensure everything was correct. I think another issue of mine is that I try to keep the schematic clean/polished as I am wiring a part up. I think a better and more efficient workflow would be to wiring everything up first and then polish so I do not have to redo a part of a pull down resistor, for instance, it is inserted and gets in the way of a power rail.

Another thing that extended this rather quick job was changing the Schottky diode suggested in the RoboticWorx guide. As I was adding the originally suggested 1N5819 diode, I remembered I had read about how some diodes are designed to have ultra low leakage. They help save standby power by minimizing the energy wasted from current flowing back through the diode. I’m pretty sure the diode is there in the first place to help protect my computer if one decides to add a battery to it. I meant to select a diode and add it to my Google Docs but forgot so I quickly searched one up and used that diode for my devboard.
Once I was done with that, my hour of work was coming to its end so I briefly began looking into the energy harvesting PMIC, which I plan on working on tomorrow!

**UPDATE**: The BAT54 (the Schottky diode) has multiple packages labelled BAT54A, BAT54B, etc. This actually confused me at first when trying to get the KiCad symbol for it (as no symbol was provided on DigiKey). I also had to be careful to reference the right package in the datasheet (I think at one point I was incorrectly referring to the BAT54A package). Anyway, after recording I decided to look more into these packages (as they were all ultra low leakage diodes) and decided to switch the BAT54 with the dual diode, BAT54C. Still using the same footprint size, the BAT54C can protect both my USB and whatever secondary source my energy harvesting chip will use - instead of having to use two diodes. I’m going to add 10 minutes to the time. (I think it was a bit more but I don’t want to inflate time!)

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/P4Ngx1175ft-)

**Total time spent: 1.22 hours**

# November 24: Schematic Work - Power Modules

I have only alloted myself an hour to work today. I honeslty spent way to long trying to get schematics for my energy PMIC and buck regulator (although it loks like KiCAD may have hadfootprints I could already use?). Now I'm going on a scavenger hunt in the datasheets to find what I need to do. Wi;; update later!

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/mTb05KhOJIEJ)

**Total time spent: 1.05 hours**

# November 25: Schematic Work - Buck Regulator 

Today I gave myself an hour to try and complete both the buck converter and energy harvesting chip. I did complete a buck converter, but I did not complete the energy harvesting.

Yesterday, I got a little stumped on my pin assignments because I was not sure where to attach things according to the TPS62842 datasheet, however I realized after finishing my recording that I was looking at the wrong “Typical Application” diagram. The one on the front page was for a more generalized package while the package I was using had its diagram in section 9.2 of the datasheet. From there hooking up the rest of the converter was pretty simple; I found the RSET value needed for 3.3V in a table and deduced that I needed to connect my EN pin to power (as it my main system regulator and will ensure the MCU receives 3.3V power instead of the 5V taken by the USB). 

Then I went on to wiring the energy harvesting chip, which proved to be a bit more confusing. I first went through the pin assignments functions and slowly started to wire things up. I initially thought I was going to be able to wire the BQ25570 to have multiple potential applications (for instance, solar or TEG) as they looked very similar, but as I was going through it I was was noticing notable differences such as resistor values and where VOC_SAMP goes. Now I’m unsure if I should just choose on application style (I’ll likely do solar) or see if it’s worth setting up some pin jumpers to make the board more versatile. I was also a bit confused by the resistors calculations.

Also, I know the BQ25570 has its own buck converter, but I decided to use an external one as the buck converter I choose has a greater electrical tolerance. If one chooses to attach external sensors, or just use the board to transmit a continuous wifi signal, the 110 mA limit of the internal buck regulator would cause my STM32 to brown out (a new term I learned!) and reset. Using the TPS62842 is safer. (Also I think there are some complications with plugging in USB 5V into one of the BQ25570 power pins.)

Since I didn’t really want to do more searching at the time I just decided to start doing the final polish of my schematic for the last 10 minutes.
I’ll look over the datasheet, clarify my uncertainties and make an update tomorrow, hopefully with the energy harvesting PMIC done.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/1MpFG-VPIZqp)

**Total time spent: 1.12 hours**

# November 26: Schematic Work - Buck Regulator 

I will update with a more detailed log; one thing is for certain though: the schematic will be completed tommorow.

[**Laspe Recording**]()

**Total time spent: 1.03 hours**



