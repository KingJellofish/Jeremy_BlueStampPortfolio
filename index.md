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
#include <math.h>

Servo base[7];
Servo shoulder[7];
Servo elbow[7];

const int servoPowerEnableGroup1 = A15;
const int servoPowerEnableGroup2 = A14;

const int base_pins[7] = {0, 22, 25, 28, 39, 36, 33};
const int shoulder_pins[7] = {0, 23, 26, 29, 38, 35, 32};
const int elbow_pins[7] = {0, 24, 27, 30, 37, 34, 31};

const int group1[3] = {1, 3, 5};
const int group2[3] = {2, 4, 6};

const int legMountAngle[7] = {0, 34, 6, -25, 147, 176, -150};
const int base_sign[7] = {0, 1, 1, 1, -1, -1, -1};

const int base_angle_us = 1500;
const int base_swing_us = 200;
const int shoulder_angle_us = 1500;
const int shoulder_swing_us = 200;
const int elbow_angle_us = 1500;
const int elbow_swing_us = 200;

const int step_delay_ms = 100;

float direction = 0.0;
float spin = 0.0;
float speed = 0.0;

int findClosestAngle(int mountAngle, int targetAngle) {
    return mountAngle + ((targetAngle - mountAngle) > 180 ? 360 : 0);
}

float strideProjection(int legIndex, float direction, float spin) {
    int theta = radians(direction);
    int mount = radians(findClosestAngle(legMountAngle[legIndex], (int) direction));

    // unit vector for direction of travel
    float dx = cos(theta);
    float dy = sin(theta);

    // tangent unit vector to leg pivot
    float tx = -sin(mount);
    float ty = cos(mount);

    // dot product to project tangent vector onto stride vector
    float displacement_deci = dx * tx + dy * ty;

    // add optional spin for turning while walking
    return displacement_deci + spin;
}

void commandBase(int legIndex, float normalizedProjection) {
    normalizedProjection = constrain(normalizedProjection, -1.5, 1.5); // to allow for spin
    int us = base_angle_us + (int) (base_sign[legIndex] * normalizedProjection * base_swing_us);
    base[legIndex].writeMicroseconds(us);
}

void commandShoulder(int legIndex, float normalizedProjection) {
    normalizedProjection = constrain(normalizedProjection, -1.0, 1.0);
    int us = shoulder_angle_us + (int) (normalizedProjection * shoulder_swing_us);
    shoulder[legIndex].writeMicroseconds(us);
}
/*
void commandElbow(int legIndex, float normalizedProjection) {
    normalizedProjection = constrain(normalizedProjection, -1.0, 1.0);
    int us = elbow_angle_us + (int) (normalizedProjection * elbow_swing_us);
    elbow[legIndex].writeMicroseconds(us);
}
*/
void commandElbow(int legIndex) {
    elbow[legIndex].writeMicroseconds(elbow_angle_us);
}

void homeAll() {
    for (int i = 1; i <= 6; i++) {
        base[i].writeMicroseconds(base_angle_us);
        shoulder[i].writeMicroseconds(shoulder_angle_us);
        elbow[i].writeMicroseconds(elbow_angle_us);
    }
}

void walkDirection(float direction, float spin, float speed) {
    const int* liftGroup;
    const int* standGroup;

    for (int group = 0; group < 2; group++) {
        liftGroup = (group == 0) ? group1 : group2;
        standGroup = (group == 0) ? group2 : group1;

        // reset previous group (no delay)
        for (int i = 0; i < 3; i++) {
            base[standGroup[i]].writeMicroseconds(1500);
            elbow[standGroup[i]].writeMicroseconds(1500);
        }

        // lift shoulder group
        for (int i = 0; i < 3; i++) {
            commandShoulder(liftGroup[i], 1.0);
        }
        delay(step_delay_ms);

        // bases forward
        for (int i = 0; i < 3; i++) {
            commandBase(liftGroup[i], -strideProjection(liftGroup[i], direction, spin) * speed);
        }
        delay(step_delay_ms);

        // plant shoulders
        for (int i = 0; i < 3; i++) {
            commandShoulder(liftGroup[i], -0.3);
        }
        delay(step_delay_ms);

        // pull bases back
        for (int i = 0; i < 3; i++) {
            commandBase(liftGroup[i], strideProjection(liftGroup[i], direction, spin) * speed);
        }
        delay(step_delay_ms);
    }
}

unsigned long start;

void setup() {
    pinMode(servoPowerEnableGroup1, OUTPUT);
    digitalWrite(servoPowerEnableGroup1, HIGH);
    pinMode(servoPowerEnableGroup2, OUTPUT);
    digitalWrite(servoPowerEnableGroup2, HIGH);
    //pinMode(LED_BUILTIN, OUTPUT);

    for (int i = 1; i <= 6; i++) {
        base[i].attach(base_pins[i]);
        shoulder[i].attach(shoulder_pins[i]);
        elbow[i].attach(elbow_pins[i]);
    }

    homeAll();
    delay(1000);

    Serial.begin(9600);
    Serial.println(F("Instructions: Send direction in degrees from -179 to 180. Send optional spin/speed as such: spin/speed:x where x is a float. To stop, send 'stop'."));
}

void loop() {
    if (Serial.available()) {
        String line = Serial.readStringUntil("\n");
        line.trim();
        if (line.length() > 0) {
            if (line.equalsIgnoreCase("stop")) {
                speed = 0.0;
            } else if (line.startsWith("spin:")) {
                spin = line.substring(5).toFloat();
            } else if (line.startsWith("scale:")) {
                speed = line.substring(6).toFloat();
            } else {
                direction = line.toFloat();
                speed = 1.0;
            }
            Serial.print(F("Direction: "));
            Serial.print(direction);
            Serial.print(F(", Spin: "));
            Serial.print(spin);
            Serial.print(F(", Speed: "));
            Serial.println(speed);
        }
    }

    if (speed > 0.001) {
        walkDirection(direction, spin, speed);
    } else {
        homeAll();
        delay(200);
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
