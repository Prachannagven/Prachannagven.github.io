# Getting Started with the Nucleo N6
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
"The STM32N6 chip is based on the ARM Cortex-M55 processor, with the Arm TrustZone and FPU ." (UM3249)

This is basically the reason we have so many problems even getting started with this chip. A normal STM32 that you may have used before simply takes your program, puts it in flash, and happily executes it from it's start point (usually 0x0800 0000). In the case of the N6 however, every single program has to boot through a trusted bootchain.

To that effect, the program consists of 3 parts:
1. First Stage Bootloader (FSBL)
2. Secure Application
3. Non-Secure Application
At poject creation time, you'll see all of these exist, based on how you pick your settings.

The boot process is as follows:
1. Boot ROM (internal to chip) copies the FSBL binary from external memory into internal SRAM.
2. The chip then jumps to the FSBL program.
3. FSBL then configures the binaries and sets timers and the like
4. FSBL then copies in the main application into the main SRAM, or instructs the chip to read from external flash.

The articles I've listed above go into more detail, but for the purposes of this article, it's sufficient to understand that the FSBL is the first piece of code that gets running after an N6 starts up.

# The Actual Blinky
1. Begin by opening up CubeMX and creating a new project using the Nucleo-N6 board. You'll be greeted by a screen that looks like below:
![STM32CubeMX Screen for N6](/assets/getting-started-with-nucleo-n6/cube-mx-open.png)
2. Go to `System Core > GPIO` and make sure all the pins are Context Assigend to the FSBL.
![](/assets/getting-started-with-nucleo-n6/incorrect-cubemx-gpio-config.png)
![correct](/assets/getting-started-with-nucleo-n6/correct-cubemx-gpio-config.png)
3. Once you've set that, go to the Project Manager tab, and make sure only FSBL is ticked in Project Structure
4. Now open the project in your preferred editor. I'm going with STM32CubeIDE for simplicity, although I'll be migrating to NeoVim soon.
5. Update the user part of your while block in main.c to have a simple binary counter using the 3 user LEDs:
``` C   
/* USER CODE BEGIN WHILE */
  uint8_t count = 0;
  while (1)
  {

	  if(count == 7){
		  count = 0;
	  }
	  else {
		  count = count + 1;
	  }

	  HAL_GPIO_WritePin(GPIOG, GPIO_PIN_8, (!(count & 0x01) ? GPIO_PIN_SET : GPIO_PIN_RESET));
	  HAL_GPIO_WritePin(GPIOG, GPIO_PIN_0, (!(count & 0x02) ? GPIO_PIN_SET : GPIO_PIN_RESET));
	  HAL_GPIO_WritePin(GPIOG, GPIO_PIN_10, (!(count & 0x04) ? GPIO_PIN_SET : GPIO_PIN_RESET));

	  HAL_Delay(1000);
    /* USER CODE END WHILE */
```
6. Build the code. **Do not debug/flash it yet**
7. Open up the terminal in your Debug folder. 
8. Type in the following command to sign the generated `.bin` file and prepare it for flashing.
`<PATH_TO_STM32CUBEIDE>/STM32CubeProgrammer/bin/STM32_SigningTool_CLI -bin <NAME_OF_PROJECT>_FSBL.bin -nk -of 0x80000000 -t fsbl -o Project-trusted.bin -hv 2.3 -dump Project-trusted.bin -align`
9. The above command generates a `Project-trusted.bin` which can finally be flashed to the system. You should see an output like below:

```
      -------------------------------------------------------------------
                       STM32 Signing Tool v2.22.0
       -------------------------------------------------------------------

Do you want to replace this file: Project-trusted.bin ? (y/n)
y
[Header v2.3] Align the payload to the 0x400 offset by adding padding bytes at the beginning of the payload.
Adding head padding bytes for payload with header v2.3
 Header version 2.3 preparation...
Extracting Entry point value from the input file...
Entry point value : 0x34180fb9
 The headred image file generated successfully:  Project-trusted.bin

Header description:

    Magic: 0x53544d32
    Signature: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
               00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
               00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
    Checksum: 0x41b91e
    Header version: 0x20300
    Size: 0x9780
    Load address: 0xffffffff
    Entry point: 0x34180fb9
    Image version: 0x0
    Extension: 0x80000000

Pad header detected:
    Type: 0x5354ffff
    Size: 0x1a0
    Padding values: f0 cf c7 2d c9 bc d4 f1 76 ef c8 4d 76 39 d7 2e 7e 0d ff 23 cc 21 7d 1a 02 18 09 1e 66 ef 88
                    50 92 d3 24 b8 aa 68 0d e0 a6 ad 76 a3 14 5b 91 91 d0 bf 8f 07 c2 3a 72 4b 08 f7 96 21 15 7e
                    b7 19 8d 1f 35 bd e4 e1 ba dc f2 47 f5 26 c5 00 f3 b6 dd be a9 d9 2d f3 22 0a 3a 09 21 21 2f
                    a1 03 f6 97 32 72 9b 5b 08 1b 18 af 46 06 b1 6d 89 3d 76 ed ff b8 49 ab fd 24 c6 70 12 5c 40
                    48 b9 bf d7 da f4 77 28 01 dc f3 ce aa 57 fa 4c 17 cc 85 bc e7 b0 2f 3e e8 10 79 c7 e5 01 16
                    ef 55 3c 6c ea 68 dd d5 b4 de dd 82 c1 53 b8 28 71 ee 14 b6 1e 25 8c 4e 35 78 1b cc e2 47 61
                    9b 91 10 ef 0e 70 a4 39 63 bf 15 38 57 78 95 f9 fe 35 f6 a9 5c d6 55 3c 76 93 03 20 90 7c c9
                    29 0b 2c ab 94 f6 ff ba 29 dd d1 34 92 1b 52 6b d8 a1 26 27 8a fe aa 10 f3 41 b8 86 93 f8 3b
                    87 f3 25 80 71 1b 73 82 5b 4d d1 52 06 2b 8a 5f 01 64 f1 ed 73 87 63 cd 6c c1 e3 78 da 43 33
                    75 d7 fc 33 67 23 44 f5 5d 2a 71 3d 59 00 3d 2b 25 45 61 fa 0e cd 46 45 e3 80 43 32 e4 47 f5
                    c3 54 04 e9 58 0c 36 44 3c 06 69 e4 b6 46 5f 51 02 b0 63 65 a5 98 ef e8 30 70 60 a2 c3 b2 db
                    18 eb f0 61 b4 fd 9a 9c 9e de b9 21 80 c1 c3 7a 66 43 6d 41 e5 f3 e7 05 3e 54 7b cb 16 62 c6
                    bc b2 6b 2c 6e 4e 6e 39

Post header 2.3 information :
    Binary type: 0x10
    Non authenticated payload length: 0x0
    Non authenticated payload hash: 0x0
```
This is essentially a header that is added onto the FSBL image. It allows the internal bootloader to authenticate the overall image, and actually allows the program to be accepted by the chip.
10. The default configuration of the jumpers (out of the box) must now be changed. Specifically, Boot1 must now be set to high. This sets the board in `Develpment Boot Mode`, allowing us to flash our firmware.
<img src="/assets/getting-started-with-nucleo-n6/default_config.jpg">
![Correct Config for Programming](/assets/getting-started-with-nucleo-n6/correct_configuration.jpg)
11. Open up CubeProgrammer, and then select the extended loader button and pick the appropraite one.
![Extended Loader](/assets/getting-started-with-nucleo-n6/cubeprogrammer-selecting-el.png)
12. Now just go to the download section, select the authenticated file, and then download it (after selecting verify).
![Download Screen](/assets/getting-started-with-nucleo-n6/cubeprogrammer-programming.png)
13. Now just go to the nucleo board, change Boot1 back to low, and then reset the board. Your program should be successfully running now!
