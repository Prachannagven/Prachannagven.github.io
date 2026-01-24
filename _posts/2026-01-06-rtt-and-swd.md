# No Debug Messages for You!
Something that I realized I took for granted when working on normal boards and programming them was debug messages. If something went wrong, or if the code stopped working, I could always see it. However, this is surprisingly harder to implement than you'd expect.

# The Process
There are two main ways of getting debug messages over SWD:
    1. *The SWO Pin* - An optional pin that most debuggers have, but most boards don't provide that pin, especially the ones that I'm working on.
    2. *RTT* - Real Time Transfer is a protocol developed by Segger, for use with their Segger J-Link Debuggers.

As is pretty obvious, I'm going to be using the RTT protocl. However, the first problem was procuring a Segger J-Link debugger. This is mainly because they're *expensive*. However, I got my hands on an NRF52840DK, which has a J-Link debugger on board. 

## The Hardware Setup

