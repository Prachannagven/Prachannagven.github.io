# Introduction
The STM32N6 chip has been pretty hyped up over the past year or so, for it's [Neural-Art](https://www.st.com/resource/en/product_presentation/st-neural-art-accelerator-introduction.pdf) accelerator (and thus ability to put neural networks on it). 

I decided I wanted to try it out myself. While the standard demo is done with their [STM32N6 Discovery Kit](https://www.st.com/en/evaluation-tools/stm32n6570-dk.html), it was really expensive, especially for just fooling around. Luckily, the [NUCLEO-N6](https://www.st.com/en/evaluation-tools/nucleo-n657x0-q.html) provided a relatively low cost alternative.

After finally getting my hands on one (try Mouser or Evalta for yours), I opened it up, plugged it in, and that's where my problems started. To the best of my knowledge, there's no clear guide on using the Nucleo-N6 - most of the documentation is for the N6-DK, so I'm writing this in the hope that I'll be able to track my progress using this diabolical board.

# Preliminary Errata
Read this before going any further. The bringup of the firmware for this board is dependent on the software you use.

## Required Software
Here's a list, I've detailed the specs further. 
- STM32CubeIDE
- STM32CubeProgrammer
- STM32CubeMX
- Most any flavor of linux that can run this

### STM32CubeIDE 
v1.17 - v1.19 provide you with the ioc generator within the setup itself. This is pretty nice, so you can go with those. However, I wanted to stick to the new toolchain released by ST, so I used **v2.0.0**. It really doesn't matter thoug, as long as it's more than v1.17.0

### STM32CubeProgrammer
Nearly any version will work for this, but I think the minimum allowed is v2.18. I personally used **v2.20**.

### STM32CubeMX
Anything greater than 6.13 will work. For this demonstration, I used **6.17.0**.

## Reference Material and Documentation
I went through quite a few documents to figure this board out, since it was my first time working with the TrustZone with STM32 MCUs. I'm listing everything I used here, although you definitely don't have to go through them all.
- [UM3249 - Getting Started with N6](https://www.st.com/resource/en/user_manual/um3249-getting-started-with-stm32cuben6-for-stm32n6-series-stmicroelectronics.pdf) - This was just my starting point. I skimmed through to find the TrustZone requirements and steps, as well as to figure out the overall architecture of the chip (roughly).
- [UM3417 - STM32N6 Nucleo-144 Board](https://www.st.com/resource/en/user_manual/um3417-stm32n6-nucleo144-board-mb1940-stmicroelectronics.pdf) - To get the LED pin numbers, general mechanical and electrical specifications, as well as required jumper setup.
- [AN5394 - Getting Started with TrustZone Projects](https://www.st.com/resource/en/application_note/an5394-getting-started-with-projects-based-on-the-stm32l5-series-in-stm32cubeide-stmicroelectronics.pdf) - Technically speaking, this is only for the L5 series, but UM3249 links this as a guide for using TrustZone in even the N6 series chips. Section 2.2.3 covers it in more detail, and gave me a starting point for the options I had to select in CubeMX.
- [UM3300 - STM32N6 Discovery Kit](https://www.st.com/resource/en/user_manual/um3300-discovery-kit-with-stm32n657x0-mcu-stmicroelectronics.pdf) - This was more as reference. I translated some guides from the DK to the Nucleo, so I needed this handy to figure out changes in Boot config, power pins, etc. Mostly useless here, but it's worth a look if some guide on the DK exists, and you wanna reference.
- [Nucleo-N6 CAD Files](https://www.st.com/en/evaluation-tools/nucleo-n657x0-q.html#cad-resources) - Used this to import the main PCB of the Nucleo so that I could validate pin connections and the like. I personally imported it to KiCad, but you can probably use any software you want, as long as it can import Altium Designer files.
- [A Somewhat Helpful Question](https://community.st.com/stm32-mcus-boards-and-hardware-tools-26/nucleo-n657x0-q-failed-to-start-gdb-server-144705) - This was actually where I started, and I traversed through ST's pages figuring this out.
- [A Somewhat Actually Helpful Guide, but Only For the DK](https://community.st.com/stm32-mcus-60/how-to-create-an-stm32n6-fsbl-load-and-run-146359) - This guide helped me understand the secure region and other stuff. Good read.
- [RM0468 - Reference Manual for Boot Modes](https://www.st.com/resource/en/reference_manual/rm0468-stm32h723733-stm32h725735-and-stm32h730-value-line-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) - This is big, and you don't really need this.
Wow that's a lot. But it is everything. You don't need to read it all, but perhaps have it on hand if you need it. With that out of the way, let the problems begin!

# A Brief Explanation of how this Chip Works
The STM32N6 chip is based on the ARM Cortex-M55 processor, with the Arm TrustZone and FPU (UM3249). This basically 
