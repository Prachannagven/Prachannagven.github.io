# Developing for the nRF52810 Chip

After playing with STM32, I wanted to go back to the nRF chip. I **hate** Zephyr. The learning curve is absolutely horrible, and I can't deal with it.

So, I'm going back to nRF5 SDK. A beautiful nice SDK built in C. I'm presuming to be working with the [nrf52810 module by ISC](https://evelta.com/nrf52810-multiprotocol-ble-ant-2-4ghz-module/). I have to design a PCB or something to work with it I guess, but for now I'll just code for the nrf52810.

The thing is, the SDK isn't really defined for the nrf52810, and most examples aren't made aware for it. However, we basically modify the flags for the pca10040 to work it out. You can look at the [Developing for the nRF52810](https://docs.nordicsemi.com/r/bundle/nrf5_sdk_v17.1.0/page/nrf52810_user_guide.html) chip on the page. You basically:
- Make sure the linker script has the right ROM and RAM end points
- Drop the `DEVELOP_IN_NRF52832` and `NRFX_COREDEP_DELAY_US_LOOP_CYCLES` flags in the Makefile inside armgcc.

So, I basically set up the project that way. It wasn't too hard. 
