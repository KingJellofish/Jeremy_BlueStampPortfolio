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

<iframe width="560" height="315" src="https://www.youtube.com/embed/l4NmR7XJLtE?si=4x3Q3vZmPOnKehZ1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
My first milestone was completing the mechanical assembly of the hexapod. It has 6 legs, each with 3 servos used for a base, a shoulder, and an elbow. I can run code that uses these servos in conjunction to move the robot forward. It can move itself forward on 6 legs, mimicking the way an ant walks, or it can crab walk (walking on the four corner legs while holding up its two middle legs). It can also sit and wave its two middle arms.

I faced challenges configuring the servos to their home positions, but I circumvented this issue by attaching the servo attachments to the servo horns at appropriate angles while they were powered. With this, the servos move to their appropriate positions within a small margin of error.

Currently, I'm facing challenges with the power provided to the servos. With the USB cable connected to my laptop, the servos move at the correct speed. Without it, the servos move a tiny amount every couple of seconds. I need to reexamine the schematic to see why this is happening. After that, I'll get some other basic movement functions operating, and then I can start adding my modifications.

I hope to add a camera and a Raspberry Pi processor that can send commands to the Arduino microcontroller, i.e. waving when the camera recognizes a human or a pet dog. Additionally, I would like to add a speaker, a microphone, and an LED display, so that the robot has more stimuli and responses. I'll calculate the required voltage for each of these modifications and see whether my current batteries source enough current for all of them to run simultaneously.

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

void walkForward() {
    // *** first trio ***

    // shoulders up (and reset other shoulders)
    shoulder2.writeMicroseconds(1500);
    shoulder4.writeMicroseconds(1500);
    shoulder6.writeMicroseconds(1500);

    shoulder1.writeMicroseconds(1300);
    shoulder3.writeMicroseconds(1300);
    shoulder5.writeMicroseconds(1700);
    delay(200);

    // rotate forward
    base1.writeMicroseconds(1300);
    base3.writeMicroseconds(1300);
    base5.writeMicroseconds(1700);
    delay(200);

    // shoulders down
    shoulder1.writeMicroseconds(1700);
    shoulder3.writeMicroseconds(1700);
    shoulder5.writeMicroseconds(1300);
    delay(200);

    // rotate backward
    base1.writeMicroseconds(1500);
    base3.writeMicroseconds(1500);
    base5.writeMicroseconds(1500);
    delay(200);

    // *** second trio ***

    // shoulders up (and reset other shoulders)
    shoulder1.writeMicroseconds(1500);
    shoulder3.writeMicroseconds(1500);
    shoulder5.writeMicroseconds(1500);

    shoulder2.writeMicroseconds(1300);
    shoulder4.writeMicroseconds(1700);
    shoulder6.writeMicroseconds(1700);
    delay(200);

    // rotate forward
    base2.writeMicroseconds(1300);
    base4.writeMicroseconds(1700);
    base6.writeMicroseconds(1700);
    delay(200);

    // shoulders down
    shoulder2.writeMicroseconds(1700);
    shoulder4.writeMicroseconds(1300);
    shoulder6.writeMicroseconds(1300);
    delay(200);

    // rotate backward
    base2.writeMicroseconds(1500);
    base4.writeMicroseconds(1500);
    base6.writeMicroseconds(1500);
    delay(200);
}

void crabWalkForward() {
    shoulder2.writeMicroseconds(544);
    shoulder5.writeMicroseconds(2400);
    elbow2.writeMicroseconds(2100);
    elbow5.writeMicroseconds(844);
    // *** first duo ***

    // shoulders up (and reset other shoulders)
    shoulder3.writeMicroseconds(1500);
    shoulder4.writeMicroseconds(1500);

    shoulder1.writeMicroseconds(1300);
    shoulder6.writeMicroseconds(1700);
    delay(200);

    // rotate forward
    base1.writeMicroseconds(1300);
    base6.writeMicroseconds(1700);
    delay(200);

    // shoulders down
    shoulder1.writeMicroseconds(1700);
    shoulder6.writeMicroseconds(1300);
    delay(200);

    // rotate backward
    base1.writeMicroseconds(1500);
    base6.writeMicroseconds(1500);
    delay(200);

    // *** second duo ***

    // shoulders up (and reset other shoulders)
    shoulder1.writeMicroseconds(1500);
    shoulder6.writeMicroseconds(1500);

    shoulder3.writeMicroseconds(1300);
    shoulder4.writeMicroseconds(1700);
    delay(200);

    // rotate forward
    base3.writeMicroseconds(1300);
    base4.writeMicroseconds(1700);
    delay(200);

    // shoulders down
    shoulder3.writeMicroseconds(1700);
    shoulder4.writeMicroseconds(1300);
    delay(200);

    // rotate backward
    base3.writeMicroseconds(1500);
    base4.writeMicroseconds(1500);
    delay(200);
}

void wave() {
    base1.writeMicroseconds(1500);
    base2.writeMicroseconds(1500);
    base3.writeMicroseconds(1500);
    base4.writeMicroseconds(1500);
    base5.writeMicroseconds(1500);
    base6.writeMicroseconds(1500);

    shoulder1.writeMicroseconds(1500);
    shoulder3.writeMicroseconds(1500);
    shoulder4.writeMicroseconds(1500);
    shoulder6.writeMicroseconds(1500);

    elbow1.writeMicroseconds(1500);
    elbow3.writeMicroseconds(1500);
    elbow4.writeMicroseconds(1500);
    elbow6.writeMicroseconds(1500);

    shoulder2.writeMicroseconds(544);
    shoulder5.writeMicroseconds(2400);

    // back
    elbow2.writeMicroseconds(2400);
    elbow5.writeMicroseconds(544);
    delay(200);
    
    // and forth
    elbow2.writeMicroseconds(2000);
    elbow5.writeMicroseconds(944);
    delay(200);
}

void strafeRight() {
    // *** first trio ***

    // shoulders up (and reset elbows and other shoulders)
    elbow1.writeMicroseconds(1500);
    elbow3.writeMicroseconds(1500);
    elbow5.writeMicroseconds(1500);

    shoulder2.writeMicroseconds(1500);
    shoulder4.writeMicroseconds(1500);
    shoulder6.writeMicroseconds(1500);

    shoulder1.writeMicroseconds(1300);
    shoulder3.writeMicroseconds(1300);
    shoulder5.writeMicroseconds(1700);
    delay(200);

    // elbows out
    elbow1.writeMicroseconds(1700);
    elbow3.writeMicroseconds(1700);
    elbow5.writeMicroseconds(1300);
    delay(200);

    // shoulders down
    shoulder1.writeMicroseconds(1700);
    shoulder3.writeMicroseconds(1700);
    shoulder5.writeMicroseconds(1300);
    delay(200);

    // elbows in, other elbows out, pull
    elbow1.writeMicroseconds(1500);
    elbow3.writeMicroseconds(1500);
    elbow5.writeMicroseconds(1500);

    elbow2.writeMicroseconds(1000);
    elbow4.writeMicroseconds(2000);
    elbow6.writeMicroseconds(2000);
    delay(200);

    // *** second trio ***
    
    // shoulders up (and reset elbows other shoulders)
    elbow2.writeMicroseconds(1500);
    elbow4.writeMicroseconds(1500);
    elbow6.writeMicroseconds(1500);

    shoulder1.writeMicroseconds(1500);
    shoulder3.writeMicroseconds(1500);
    shoulder5.writeMicroseconds(1500);

    shoulder2.writeMicroseconds(1300);
    shoulder4.writeMicroseconds(1700);
    shoulder6.writeMicroseconds(1700);
    delay(200);

    // elbows out
    elbow2.writeMicroseconds(1700);
    elbow4.writeMicroseconds(1300);
    elbow6.writeMicroseconds(1300);
    delay(200);

    // shoulders down
    shoulder2.writeMicroseconds(1700);
    shoulder4.writeMicroseconds(1300);
    shoulder6.writeMicroseconds(1300);
    delay(200);

    // elbows in, other elbows out, pull
    elbow2.writeMicroseconds(1500);
    elbow4.writeMicroseconds(1500);
    elbow6.writeMicroseconds(1500);

    elbow1.writeMicroseconds(1000);
    elbow3.writeMicroseconds(1000);
    elbow5.writeMicroseconds(2000);
    delay(200);
}

unsigned long start;

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

    shoulder1.attach(23);
    shoulder2.attach(26);
    shoulder3.attach(29);
    shoulder4.attach(38);
    shoulder5.attach(35);
    shoulder6.attach(32);

    elbow1.attach(24);
    elbow2.attach(27);
    elbow3.attach(30);
    elbow4.attach(37);
    elbow5.attach(34);
    elbow6.attach(31);

    start = millis();
}

void loop() {
    unsigned long now = millis();
    /*if (now - start > 25000) {
        //wave();
    } else if (now - start > 15000) {
        //strafeRight();
    } else */if (now - start > 5000) {
        strafeRight();
    } else {
        base1.writeMicroseconds(1500);
        base2.writeMicroseconds(1500);
        base3.writeMicroseconds(1500);
        base4.writeMicroseconds(1500);
        base5.writeMicroseconds(1500);
        base6.writeMicroseconds(1500);
        
        shoulder1.writeMicroseconds(1500);
        shoulder2.writeMicroseconds(1500);
        shoulder3.writeMicroseconds(1500);
        shoulder4.writeMicroseconds(1500);
        shoulder5.writeMicroseconds(1500);
        shoulder6.writeMicroseconds(1500);

        elbow1.writeMicroseconds(1500);
        elbow2.writeMicroseconds(1500);
        elbow3.writeMicroseconds(1500);
        elbow4.writeMicroseconds(1500);
        elbow5.writeMicroseconds(1500);
        elbow6.writeMicroseconds(1500);
    }
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
