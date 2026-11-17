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

**Total time spent: 1.65 hours**

# November 16: Schematic Work - Power Supply and LSE

Did a work session today where I finished up the Power Supply wiring I started yesterday. Before I began I did some searching on Google regarding how to read the datasheet properly, and so I was no longer confused on what capacitors I needed to place. I think what got me so confused yesterday was that fact I was too focused on trying to match the power supply scheme to the one in my guides, when they had notably different designs. I now understand that when the datasheet reads "n x 100 nF + 1 x 10 μF", for instance, it is saying that a `n` amount of 100nF ceramic capacitors (that being each instance of the associated pin - VDD, in this case - as not every STM32U575xx microcontroller has the same amount of pins) should be placed so each pin has its own capacitor. In addtion, a 10μF bulk capacitor should be placed, and this capacitor is shared among all the VDD pins. It was also when I was doing this that I learned it is standard to add a 100nF capacitor to VBAT (even if the power scheme did not show it) if I am not adding a seperate coin cell to my board, and that adding a ferrite bead to VDAA can create cleaner ADC sensor reading (I actually discovered this when as I was making the schematic when I was checking over my design to make sure it was electrically correct).

When I was doing research yesterday, I learned that I don't need to add a 8MHz crystal (contrary to the guides I was following) as the U5 series, being newer, has a sufficent internal oscillator designed to be used with the USB. However, I did still had to add Low Speed External clock for low power operation (which is the main focus of my devboard.) So after finishing my power scheme I decided I would try and add that too. Honeslty, it took more time finding hwat crystal I wanted and getting its schematic into KiCad than actually wiring it. The wiring was pretty simple as I had both the datasheet and guides to reference. For my load capacitance calculation (C = 2(CL - Cparasitic)) I did `2*(7pF - 3pF)`.

I did spend a bit more time than expected on making the schematic look nice, but I really want to make sure my schematic looks organized. I used Fabricate's images as a starting point and then continued adjusting for clarity by myself. I know communication is important, and my inital wiring (which I used to get my bearing on which compnents went where) was a lot harder to read upon first glance. 

Tommorow I will either continue RoboticWorx guide's flow and starting breaking out the MC's GPIOs or I will hook up the USB, SWD debugging and maybe my energy harvesting system.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/xZ3T5_3_5pFG)

**Total time spent: 1.27 hours**

