# Big Red Hacks 2019

#### 22 Sept 2019

I participated in Cornell's Big Red Hacks hackathon this year. It was fun, but we never got our project to work. The whole experience was a vivid demonstration of how hardware has gotten more integrated and thus less hackable over time.

The[ devpost project page](https://devpost.com/software/mousinson-s) has a full write up with pictures. I am also posting a copy of the write up here so that I don't lose it to the winds of internet time.

I did this writeup at midnight after two days of hacking, and no-one on my team bothered to proof read, so please forgive any typos.

## The Idea

There are over 200,000 cases of Parkinson's disease diagnosed every year. People who suffer from this disease can have a great deal of difficulty using computer mice. The conventional solutions to this problem it to adjust mouse sensitivity settings and try to go slow. There are some software packages designed specifically to help, but they are priority and have mixed reviews.

Software is not the solution. The mouse hardware itself should correct for the tremors caused by the disease. This allows for the use of any computer without the need to install costly software, and for computers to be more easily shared.

Also optical mouse sensors are cool and we want to play with them.

## The Plan
1. Open up a mouse
2. Cut the connections between the optical sensor and the microcontroller
3. Solder jumper wires onto the sensor
4. Connect our microcontroller
5. Invent a Parkinons smoothing algorithm
6. Implement the algorithm on said microcontroller
7. Rebuild the mouse with our hardware inside

## Episode 1: The Mouse Menace
The mice were all in one piece and we needed them to not be like that. We had a collection of different mice from thrift stores, amazon, eBay, and walmart.  We disassembled half a dozen and found four unique optical sensors between them. The internet told us that it is easy to hack into optical sensors. The internet lied.

One of the sensors had an easy to find data sheet and that said USB integrated into the same package as the sensor. This would would not work for us because we did not want to decode USB. Another we could tell had USB integrated in just by looking at it.

The last two did not have data sheets that were showing up on google. We began searching high and low for them. We went to the second page of google, we searched related part numbers, we reached out to vendors on AliExpress asking for data sheets. We got nothing. Almost.

On stack overflow someone was looking for the same data sheet as us, the  FCT3065-XY, and the accepted answer was that the data sheet was priportary and not on the internet. Someone left a comment however saying that the chip was a clone of the ADNS-2620, which has a data sheet and is an I2C device perfect for what we wanted to do we. We started to run with this idea.

## Episode 2: Attack of the Clock
It is not smart to trust a stranger on the internet when they say two chips from (probably) different manufactures and no apparent relation work the same. We decided to verify, and this was our first step on the path to madness.

We connected a saleae logic analyzer to the supposed clock and SDA pins on the chip and powered the mouse. When we moved the mouse the mouse cursor moved and data showed up from the logic analyzer. When we looked closer though we started to see problems. The clock line looked like trash and we couldn't decode the data going over the wire because of it. 

At first when we looked at the clock we didn't see a 50% duty cycle like the I2C standard calls for. That was bad. As we kept looking though things got worse. When the clock when high it still was showing the same clock frequency at a smaller magnitude. That is not how clocks work.

At this point it was clear something weird was going on, but a logic analyzer is not the tool to decode analogous problems. We first soldered jumper wires onto the chip to make sure we weren't seeing the results of a poor connection. We then connected the chip to an osociscope.

I have looked at a lot of broken hardware on a scope and I have never seen something change behavior like this. It was bizarre to the point I don't want to go through the trouble describing it. Furthermore the mouse was mousing the whole time so it wasn't even broken. We have spent so much time on this chip, it is time to cut our losses.

## Episode 3: Revenge of the Algorithm

This is where I slightly break out of the chronological storytelling. 

Throughout the hackathon we were working to develop the smoothing algorithm we use. To do this we were pulling data from `/dev/mice` and piping it into C to develop and test. This sounds trivial but `stdout` was not cooperating and pipes were being less than reliable. A lot of time was lost to BASH and the C compiler. I still don't understand how `stdout` can break.

We did have success on the algorithm front though. When we got some test data we plotted in through python. We then use python to apply a low pass filter and b-spline to the data. This should promising results in taking out high to medium frequency hand tremors. It was also lightweight enough that we could implement it on a microcontroller and run it in real time.

The revenge was that we could never fully test it without hardware.

## Episode 4: A New Hope

It is a period of despair. 

Google has failed. Large corporations hide datasheets away from us. Clocks are no longer periodic.

Pursued by the hackathon's sinister agents, we races home, custodian of one last piece of hardware that can save us and restore freedom to the galaxy....

The ADNS-9800 Laser Motion Sensor

Planning for the worst we order a breakout board for this sensor from Tindie. It is designed to be used by hackers. It is well documented. Code libraries exist. It will save us from the failures of true hardware hacking.

## Episode 5: The Sensor Strikes Back

It should be easy. We follow the instructions by the board designer to put the board in 5V mode. We wire it into our arduino pro mice. We try to compile and upload the example code.

The code does not compile.

It's okay, it was written for an older version of Arduino. We change some variables from `prog_uchar` to `unsigned char`, and the code compiles. 

We upload it, open the serial monitor, and don't see anything.

It's alright. We are using the Arduino Pro Micro because it is based on a ATmega32U4. That chip has native USB so it can appear as a mouse to a computer, but it is not the standard arduino chip. Let's connect the sensor to an arduino Uno, which is the most common arduino and uses an ATmega32U4. Still doesn't work.

We have another random SPI device (the protocol we are using with this optical sensor), let's make sure that it works with everything. It works perfectly with the Uno and the Micro. 

Great! I guess...

Next we check the sensor board to make sure it is working. When it is powered is should be illuminating an LED, but we don't see anything. The datasheet says that the LED is at 820nm wavelength, that's IR. We can't see it but some cameras can, we start filming the LED with our phones. We don't see the LED but not all cameras record IR.

We start probing the board with a multimeter. We are inputting 5V, but only reading 3V at the chip. That's probably not good. We open the schematics for the board and find that even though the optical sensor can be powered at 5V the board always powers it at 3. It turns out that what we did to put the board in 5V mode just sends the data signals through *voltage dividers* to step them down to 3V.

That's not how you level shift.

That's not how you design hardware.

Now we know the designer of our back up plan didn't know what they were doing.  It's alright though, just because they didn't do it the right way doesn't mean it won't work. It works for other people on the internet.

We know from the logic analyzer that the SPI to the optical sensor is not working.  We know that we have other SPI that is working. We decided to take the drivers for the other chip and modify them for our chip. This should work. We have the data sheet.

After hours of troubleshooting our homespun SPI Arduino drivers compile. We uploaded it to the Pro Micro, and it bricks. We can't reset the Arduino and we can't communicate with it. This was not the first Arduino we bricked during the hackathon, but it was the last. It is time to sleep.

## Episode 6: Return of the Project

This part of the story has yet to be written. What we have now is piles of broken hardware, endless broken code, and exhaustion. 

These sarifies will not be in vain. 

We have learned so much about hardware design, communication protocols, C, smoothing algorithms, and hardware debugging. We will regroup, and we will conquer this challenge. If not at this hackathon then at the next.

## The Extended Universe

It is important to fully embrace your surroundings. Here some advice for visiting Ithaca:

* The top of the Cornell bell tower has great views, but the bells are loud
* Waffle Frolic is delicious
* College Town Bagels is a nice spot to have a drink after a long day of hacking
* Cornell has a beautiful campus. You should spend as much time walking around it as you can