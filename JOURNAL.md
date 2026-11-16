---
title: "Flint Devboard"
author: "Reuben 0"
description: "Imitating flint, this zero-power custom STM32 devboard slowly builds up potential and strikes a quick wireless spark to transmit Bluetooth or Wi-Fi packets." 
created_at: "2026-11-15"
---
# November 15: Initial Research + Schematic Startup

I have been wanting to try and make a devboard for a while. I think it would be a good introductory project to either PCBA or soldering with a hot plate (we shall see what option I choose). 

I originally thought of following the RP2040 devboard guide on Forge, but if I did that I would just be making a devboard for the sake of it. I wanted to create something with a somewhat unique feature. So I started looking for some more “unique” devboard ideas online and stumbled on a [devboard guide](https://www.instructables.com/Custom-STM32-Boards-From-Scratch-the-Complete-Guid/) for the STM32. As I was reading through it, I learned that some STM32 boards could manage power more efficiently than the ESP32, a microcontroller I considered using for a wireless macropad but decided against it because of its high power consumption (leading to a low battery life). This inspired me to make an STM32 devboard that had equivalent functionality to an ESP32 but used much less power. To make this more unique and emphasis my board’s low power functionality, I decided I am going to make the dev board a “Zero-idle energy harvester” by adding an energy-harvesting power management IC (PMIC) and a load switch so my board becomes a maintenance free platform (can be run on tiny solar cells or thermoelectric generators). 

With my plan now made, I decided I would follow the guide I mentioned earlier while adding in my own modifications. I also found a guide on [Fabricate](https://fabricate.hackclub.com/docs/guides-devboard#doc) that I might also reference as I am making this. I made a brief list of the things I will need to change or add to my project and began to start the PCB schematic with the help of Google and also some help form its AI search overview. At this point, I finally remembered to record my progress using Laspe. During the lapse I created a new KiCAD project, added the schematic for the STM32U575RIT6 and attempted to start wiring the power system while referencing its datasheet.

[**Laspe Recording**](https://lapse.hackclub.com/timelapse/9Q6iECUoty0G)

I added an hour of time to account for the time I spent researching prior to recroding the lapse.

**Total time spent: 1.65 hours**
