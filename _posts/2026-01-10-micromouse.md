# Micromouse!
A pretty common high school/college competition is [Micromouse](https://en.wikipedia.org/wiki/Micromouse). It essentially involves building a robot that can solve a maze. Now, my friends and I tried participating in micromouse at our college last year, but 12 hours before the competition, we found that bluetooth couldn't be used while a certain pin was also used on the ESP32. We also realized double integrating acceleration, *really* didn't work to get distance.

This annoyance kinda fuelled us to participating in this year's competition. So, with a budget of about 5k (tools included), we wanted to build a micromouse that would complete the maze, and hopefully win. 

# The Setup
Following last year's abysmal performance, I decided to make a few changes this time so that we'd do better. Both hardware and (some) software side changes were made, primarily to increase accuracy, and also to help with debugging.

## Hardware
Our previous year's hardware consisted of two motors (without encoders), an motor driver, an IMU and finally an ESP32 for the brain. Needless to say, it was absolutely horrible. The motors were too slow, the IMU was too inaccurate to tell us distance moved, and the ESP32 had a borganshmorg of "gotchas" for us to get anywhere decent with it. Not to mention, something went ***pop-fizz*** about two hours before the competition.

This year, we went with some more sane components. Starting with the brain, we went with the [Raspberry Pi Pico W 2](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html). It gave us wifi, bluetooth, and enough GPIO pins and I2C lines that we'd have no problem with our sensor array.

![Raspberry Pi Pico W 2 Pinout](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRKuSKC3mDDWBBBB90UirMm3uqwB6X8IUlyJw&s)

We used three ultrasonic sensors. I picked the HC-SR04, mainly because it was cheap and easy. The other options would've been expensive, and it seemed that these would do well enough given some software effort. They allow us to measure between 4cm and 3m, so we wouldn't have much of an issue with invisiblity. Using three of the sensors, we get "sight" in the front, left and right directions. Allowing us to navigate the maze properly.

Next, the most significant upgrade from last year are two encoder-attached dc motors. As the name suggests, they have an encoder attached to the motor shaft. This lets us keep track of the exact number of rotations that the motor shaft makes over time, being much more accurate than a double integration of acceleration. 

Finally, we used the MPU6050 as an IMU, giving us rotation so we would know if a left/right turn was properly completed. 

It should be noted that the L298N board that we used provided us with a 5V output, which we pushed into the Pi's VSYS pin. From there, all sensors and peripherals were powered on 3.3V voltage levels. This was mainly because the GPIOs on the Pi only allow 3.3V, and we didn't want to damage anything. The encoders, ultrasonics and imu all got 3.3V to their voltage line, so their output was also fine.

We used a 3S, 1000mAh lipo battery. Both because it was nice and fast to charge, not too high of a voltage, and we had it laying around (it would've been insanely expensive otherwise).
