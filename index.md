# Hexapod
My project is a six-legged contraption that uses servos to move around. The legs have multiple joints using a total of 18 servos. In the future, it will have a camera/ultrasonic sensor, a speaker, and an LED display for facial expressions, as well as the ability to remotely control it.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jeremy P | Oakwood School | Electrical Engineering | Incoming Senior

![Headstone Image](Jeremy P.heic)

# Final Milestone
<!--
For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE
-->


# Second Milestone
<!--
**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**


For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 
-->

# First Milestone
<!--
**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**



For your first milestone, describe what your project is and how you plan to build it. You can include:
- An explanation about the different components of your project and how they will all integrate together
- Technical progress you've made so far
- Challenges you're facing and solving in your future milestones
- What your plan is to complete your project
-->

# Starter Project

<iframe width="560" height="315" src="https://www.youtube.com/embed/AjJmskXgcbA?si=TMgxeei5rKUD7gwA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
I built the Retro Arcade Console as my starter project. I soldered most of the components to the board besides the microcontroller, like the electrolytic capacitor, the LED grid and score displays, the power switch, and the buttons.

# Schematics 
<!--
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 
-->

# Code
<!--
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 
-->

```c++
#ifndef ARDUINO_AVR_MEGA2560
#error Wrong board. Please choose "Arduino/Genuino Mega or Mega 2560"
#endif

#include <Servo.h>

Servo base1;
Servo base2;
Servo base3;
Servo base4;
Servo base5;
Servo base6;

Servo shoulder1;
Servo shoulder2;
Servo shoulder3;
Servo shoulder4;
Servo shoulder5;
Servo shoulder6;

Servo elbow1;
Servo elbow2;
Servo elbow3;
Servo elbow4;
Servo elbow5;
Servo elbow6;

const int servoPowerEnableGroup1 = A15;
const int servoPowerEnableGroup2 = A14;

int timer = 0;
bool front = true;

int mapAngleUs(int value, int lowerBoundUs, int upperBoundUs, int angleMax) {
    long num = (long)(upperBoundUs - lowerBoundUs) * value;
    int us = lowerBoundUs + (int)((num + 45) / angleMax);
    return us;
}

void setup() {
    pinMode(servoPowerEnableGroup1, OUTPUT);
    digitalWrite(servoPowerEnableGroup1, HIGH);
    pinMode(servoPowerEnableGroup2, OUTPUT);
    digitalWrite(servoPowerEnableGroup2, HIGH);
    //pinMode(LED_BUILTIN, OUTPUT);
    base1.attach(22);
    base2.attach(25);
    base3.attach(28);
    base4.attach(39);
    base5.attach(36);
    base6.attach(33);
}

void loop() {
    base1.writeMicroseconds(mapAngleUs(55, 1400, 2400, 80));
    base2.writeMicroseconds(mapAngleUs(13, 600, 1600, 56));
    base3.writeMicroseconds(mapAngleUs(40, 600, 1300, 56));
    base4.writeMicroseconds(mapAngleUs(25, 1000, 2000, 80));
    base5.writeMicroseconds(mapAngleUs(37, 1000, 2000, 80));
    base6.writeMicroseconds(mapAngleUs(15, 1200, 2200, 80));
    delay(50);
    //digitalWrite(LED_BUILTIN, !digitalRead(LED_BUILTIN));
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Freenove Hexapod Robot Kit | Base materials and electronics | $126.99 | <a href="[https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/](https://store.freenove.com/products/fnk0031)"> Link </a> |
| Mod 1 | Fun | $??? | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Mod 2 | Fun | $??? | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

<!--
# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
-->
