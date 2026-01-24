# An IDE? What's that? (Bare Metal C Programming 1)

I am, by no means, an expert in the field of embedded electronics. However, I have tried my hand with all the major types (STM, ATMEGA, NRF, MSP) of chips, but always with their own in built editors.
So, I decided it was time to kill myself a little and dive into the world of bare-metal programming.

By definition, bare-metal programming is a method of programming wherein you directly interact with the registers of the associated microcontroller, without any intermediate abstraction layers.
In the case of the STM32, there are quite a few possible abstraction layers. Most commonly, you'll use the HAL API provided by STM for your particular microcontroller. Being a standard API, it allows you to port code between different microcontrollers across the ST series.

However, all that standardization comes at the cost of efficiency. HAL compiled code comes with a whole host of bloated additional code and problems, which may not be your style. 
While an actual developer may have a necessary use for this efficacy, I'm simply doing it out of interest.

## The Setup

### The Reference Manual
A Redditor on r/embedded suggested the book "Bare-Metal Embedded C Programming" for anyone trying their hand at this, and I had to take a look. The book comes with a [Github Repository](https://github.com/PacktPublishing/Bare-Metal-Embedded-C-Programming) which has the code and projects covered in every chapter.

I'm going to be reading through and trying my hand at implementing the projects covered in the various chapters.

### Hardware
I'm doing this on linux (obviously), running KDE Plasma 25.10, since I find it easiest to run things out of the terminal. You can obviously do it on Windows, and all the software is available for that OS too.

I'm using an STM32F103R Nucleo Board for all my hardware testing. I've listed the associated reference docs from ST below, but if you're following along, you'll obviously have to find the manuals corresponding to your board.

- [STM32 Nucelo Board User Manual](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf)
- [STM32F103RB Chip Datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)
- [ARM Cortex M3 Datasheet](https://www.arm.com/-/media/Arm%20Developer%20Community/PDF/Processor%20Datasheets/Arm%20Cortex-M3%20Processor%20Datasheet.pdf)
- [STM32F10xxx Reference Manual](https://www.keil.com/dd/docs/datashts/st/stm32f10xxx.pdf)

I'll be referring to these four documents throughout this project.

### Additional Software
I have STM32CubeIDE already installed and setup on my system. This is gonna help with comparing the code that I write to the code that is actually developed directly by ST via their IDE. 

You'll also need [OpenOCD](https://openocd.org/) to flash data directly to the chip. You'll also need a terminal viewer. I'm using telnet, but you can use pretty much anything for this.
