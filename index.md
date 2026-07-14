# Hexapod
My project is a six-legged contraption that uses servos to move around. The legs have multiple joints using a total of 18 servos. These work in conjunction with each other using inverse kinematics to calculate where to position them to most effectively walk in a direction. It can be commanded wirelessly using a remote control equipped with a joystick, potentiometers, and a few buttons. The hexapod will be equipped with a USB webcam and a Raspberry Pi capable of image processing to provide live feedback, in addition to a microphone to listen for human input. To express itself, it will also include an LED display and a speaker for live responses to stimuli.

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

<iframe width="560" height="315" src="https://www.youtube.com/embed/AKbBMT0cKrQ?si=8zjm9QAUEI72863R" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
My second milestone was enabling the hexapod to walk in any direction. After two weeks of calculations, I've successfully created adjustable settings to customize the gait, including speed, stride, direction, and how much each leg sinks into the floor while walking.

At first, I tried driving each leg based on its potential to do work in the direction of movement. I did this by calculating the dot product of the direction vector with each base servo's direction of movement, as well as each elbow servo's (they are perpendicular, which made me assume that I could achieve any direction). This proved quite ineffective, most likely because each servo traced an arc on the ground rather than a linear path for the most direct push.

After that, I pivoted to real 3 DOF inverse kinematics. Using the law of cosines, I derived each servo's angle in radians based on a point in (x,y,z) space. With this relationship, I could simply request a point in space for the tip of the leg to be at, and by mapping pulse widths in microseconds to a 0-180 degree (0-PI/2 radians) range, each servo would adjust to the correct angle to position the tip of the leg at the desired point. Then, I used linear interpolation to continuously drive each servo in the opposite direction of movement, so that they would push in that direction. When they are not pushing, they return to the start position in a sinusoidal motion.

[Add more here]

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
Hexapod Code
```c++
#ifndef ARDUINO_AVR_MEGA2560
#error Wrong board. Please choose "Arduino/Genuino Mega or Mega 2560"
#endif

// libraries
#include <Servo.h>
#include <SoftwareSerial.h>
#include <math.h>

#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

// radio for remote control, address same as the one acknolwedged in code on remote control
RF24 radio(9, 53); // CE, CSN
const byte address[6] = "00001";

// list of Servo objects for all 18 servos (plus a blank one for easy numbering)
Servo base[7];
Servo shoulder[7];
Servo elbow[7];

// servo power enable pins (built-in)
const int servoPowerEnableGroup1 = A15;
const int servoPowerEnableGroup2 = A14;

// servo pins
const int base_pins[7] = {0, 22, 25, 28, 39, 36, 33};
const int shoulder_pins[7] = {0, 23, 26, 29, 38, 35, 32};
const int elbow_pins[7] = {0, 24, 27, 30, 37, 34, 31};

// the microsecond value of each servo's home position (1472 according to angle mapping)
const int base_angle_us = 1500;
const int shoulder_angle_us = 1500;
const int elbow_angle_us = 1500;

// joystick values
double joystickX = 0.0;
double joystickY = 0.0;
int joystickZ = 0;

// where the robot wants to move
double direction = 0.0;
// how fast the legs move
double speed = 0.0;
// how far the leg goes to move
double maxStride = 0.3;
// how high the leg goes in z when returning to step again
const double returnLiftHeight = 0.5;
// how low into the ground the desired z should be when pushing
const double z_depression = 0.5;

// phase used to keep track of which tripod to be pushing and which to be swinging
double phase = 0.0;
// max amount of time incremented per loop
double maxDt = 0.02;

// length of first joint
const double b = 55.0 / 70.0;
// length of second joint
const double c = 1.0;

// angles at which each leg is mounted relative to the horizontal (measured in degrees)
const int legMountAngle[7] = {0, 34, 6, -25, 147, 176, -150};

// each leg's home position in xyz space
const double base_x[7] = {0.0, b * cos(radians(legMountAngle[1])), b * cos(radians(legMountAngle[2])), b * cos(radians(legMountAngle[3])), b * cos(radians(legMountAngle[4])), b * cos(radians(legMountAngle[5])), b * cos(radians(legMountAngle[6]))};
const double base_y[7] = {0.0, b * sin(radians(legMountAngle[1])), b * sin(radians(legMountAngle[2])), b * sin(radians(legMountAngle[3])), b * sin(radians(legMountAngle[4])), b * sin(radians(legMountAngle[5])), b * sin(radians(legMountAngle[6]))};
const double base_z[7] = {0.0, -c, -c, -c, -c, -c, -c};

// only used in forward kinematics calculations {
double theta_base[7] = {legMountAngle[0], legMountAngle[1], legMountAngle[2], legMountAngle[3], legMountAngle[4], legMountAngle[5], legMountAngle[6]};
double theta_shoulder[7] = {0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};
double theta_elbow[7] = {0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};

double x_from_thetas(int legIndex) {
    return cos(theta_base[legIndex]) * (b * cos(theta_shoulder[legIndex]) + c * sin(theta_shoulder[legIndex] + theta_elbow[legIndex]));
}
double y_from_thetas(int legIndex) {
    return sin(theta_base[legIndex]) * (b * cos(theta_shoulder[legIndex]) + c * sin(theta_shoulder[legIndex] + theta_elbow[legIndex]));
}
double z_from_thetas(int legIndex) {
    return b * sin(theta_shoulder[legIndex]) - c * cos(theta_shoulder[legIndex] + theta_elbow[legIndex]);
}
//}

// helper function for mapping radians to a pulse width
double mapThetaToUs(double theta) {
    return (theta / PI) * (2400 - 544) + 544;
}

// used to put all legs at their home position
void homeAll() {
    for (int i = 1; i <= 6; i++) {
        base[i].writeMicroseconds(base_angle_us);
        shoulder[i].writeMicroseconds(shoulder_angle_us);
        elbow[i].writeMicroseconds(elbow_angle_us);
    }
}

// a data type to store the 3 angles for a given leg
struct LegThetas {
    double base;
    double shoulder;
    double elbow;
};

// inverse kinematics function that takes a desired point in xyz and returns appropriate angles for each a leg's servos
LegThetas calculateLegThetas(int legIndex, double desired_x, double desired_y, double desired_z);
LegThetas calculateLegThetas(int legIndex, double desired_x, double desired_y, double desired_z) {
    LegThetas result;
    double x = desired_x;
    double y = desired_y;
    double z = desired_z;

    result.base = atan2(y, x);

    double r = sqrt(x * x + y * y);
    double d = sqrt(r * r + z * z);
    double alpha = atan2(z, r);
    double C = acos(constrain((b * b + d * d - c * c) / (2 * b * d), -1.0, 1.0));
    result.shoulder = alpha + C;

    double D = acos(constrain((b * b + c * c - d * d) / (2 * b * c), -1.0, 1.0));
    result.elbow = D - PI / 2;

    return result;
}

// function that commands a leg's servos to their intended angles
void writeLeg(int legIndex, LegThetas t);
void writeLeg(int legIndex, LegThetas t) {
    if (legIndex < 4) {
        base[legIndex].writeMicroseconds(mapThetaToUs(t.base + PI / 2));
        shoulder[legIndex].writeMicroseconds(mapThetaToUs(t.shoulder + PI / 2));
        elbow[legIndex].writeMicroseconds(mapThetaToUs(PI - (t.elbow + PI / 2)));
    } else {
        base[legIndex].writeMicroseconds(mapThetaToUs(PI - (t.base + PI / 2)));
        shoulder[legIndex].writeMicroseconds(mapThetaToUs(PI - (t.shoulder + PI / 2)));
        elbow[legIndex].writeMicroseconds(mapThetaToUs(t.elbow + PI / 2));
    }
}

// main walking algorithm
void walkDirection(double direction, double phase, double stride) {
    double x, y, z, time, offset;
    bool stance;
    LegThetas t;
    for (int i = 1; i <= 6; i++) {
        // phase < 0.5: ground/pushing stage for tripod1, lifting stage for tripod2
        // phase >= 0.5: ground/pushing stage for tripod2, lifting stage for tripod1
        stance = (phase < 0.5);
        time = stance ? (phase / 0.5) : ((phase - 0.5) / 0.5);
        
        if ((i % 2 == 1) != stance) { // swing back
            offset = stride * (2.0 * time - 1.0);
            z = base_z[i] + returnLiftHeight * sin(PI * time) - z_depression;
        } else { // stand and push
            offset = stride * (1.0 - 2.0 * time);
            z = base_z[i] - z_depression;
        }

        x = base_x[i] + offset * cos(direction);
        y = base_y[i] + offset * sin(direction);

        t = calculateLegThetas(i, x, y, z);
        writeLeg(i, t);
    }
}

// setup (called once)
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

    while (!Serial) {
        ;
    }

    radio.begin();
    radio.openReadingPipe(0, address);
    radio.setPALevel(RF24_PA_MIN);
    radio.startListening();
    //port.begin(115200);
}

// loop (iterated over and over)
void loop() {
    if (radio.available()) {
        char text[16] = "";
        radio.read(&text, sizeof(text));
        String line = String(text);
        line.trim(); //removes white space from joystick input
        Serial.println(line);
        
        int separatorIndex = line.indexOf(":");
        if (separatorIndex != -1) {
            String label = line.substring(0, separatorIndex);
            int value = line.substring(separatorIndex + 1).toInt();

            if (label.equals("joystickX")) {
                joystickX = value;
                speed = 1.0;
                Serial.println(joystickX);
            } else if (label.equals("joystickY")) {
                joystickY = value;
                speed = 1.0;
                Serial.println(joystickY);
            } else if (label.equals("joystickZ")) {
                //joystickZ = value;
                speed = 0;
                Serial.println(speed);
            }

            /*
            if (label.equals("joystickX") || label.equals("joystickY")) {
                speed = constrain(hypot(joystickX - 512, joystickY - 512) / 512.0, 0.0, 1.0);
            }
            */
        }
    }

    if (speed > 0.001) {
        direction = atan2(joystickY - 512, joystickX - 512);

        walkDirection(direction, phase, maxStride * speed);

        Serial.print("attempted walking: ");
        Serial.println(direction);
    } else {
        homeAll();
        delay(200);
    }

    phase += maxDt * speed;
    if (phase >= 1.0) {
        phase -= 1.0;
    }
    Serial.print("phase: ");
    Serial.println(phase);
}
```

Remote Control Code
```c++
/*
 * Sketch     Self diagnosis sketch for Remote
 * Platform   Freenove Smart Car Remote (Compatible with Arduino Uno) with 
 *            Freenove Smart Car Remote Shield and Freenove Control Board
 * Brief      This sketch is used to diagnose the remote after it has been assembled.
 *            If your remote is not working properly, follow the steps below to diagnose and fix.
 * Steps      1. Install NRF24L01 module to the remote.
 *            2. Connect remote to computer via USB cable and choose the right board and port.
 *               Then open Serial Moniter with baud 115200.
 *            4. Upolad this sketch to the remote.
 *               The Serial Moniter will show diagnostic information. 
 *               Operate the remote and the Serial Moniter will show relevant information.
 *            5. Please check the diagnostic information and try to fix the problem.
 *               Then press the RESET button to run this sketch again to see if the problem has been fixed.
 *               If yes, please upload the default sketch again to verify if the remote is working properly.
 *               If no or you can't fix the problem, please send diagnostic information and how did the 
 *               remote behave to our support team (support@freenove.com).
 * Author     Ethan Pan @ Freenove (support@freenove.com)
 * Date       2021/01/15
 * Version    V12.0
 * Copyright  Copyright © Freenove (http://www.freenove.com)
 * License    Creative Commons Attribution ShareAlike 3.0
 *            (http://creativecommons.org/licenses/by-sa/3.0/legalcode)
 * -----------------------------------------------------------------------------------------------*/

#ifndef ARDUINO_AVR_UNO
#error Wrong board. Please choose "Arduino Uno"
#endif

#include <SPI.h>
#include "RF24.h"
#include <FlexiTimer2.h>

RF24 radio = RF24(9, 10);

const byte address[6] = "00001";

bool resetX = false;
bool resetY = false;

enum InputPin { Pot1, Pot2, JoystickX, JoystickY, JoystickZ, S1, S2, S3, None };

const int pot1Pin = A0,         // define POT1
          pot2Pin = A1,         // define POT2
          joystickXPin = A2,    // define pin for direction X of joystick
          joystickYPin = A3,    // define pin for direction Y of joystick
          joystickZPin = 7,     // define pin for direction Z of joystick
          s1Pin = 4,            // define pin for S1
          s2Pin = 3,            // define pin for S2
          s3Pin = 2,            // define pin for S3
          led1Pin = 6,          // define pin for LED1 which is close to POT1 and used to indicate the state of POT1
          led2Pin = 5,          // define pin for LED2 which is close to POT2 and used to indicate the state of POT2
          led3Pin = 8;          // define pin for LED3 which is close to NRF24L01 and used to indicate the state of NRF24L01

volatile int ledState = 1;

void setup() {
  Serial.begin(115200);
  Serial.println("");
  Serial.println("");
  Serial.println("Freenove Smart Car Remote Shield and Freenove Control Board");
  Serial.println("------------------------------------------------------------------------------------------");

  pinMode(joystickZPin, INPUT);
  pinMode(s1Pin, INPUT);
  pinMode(s2Pin, INPUT);
  pinMode(s2Pin, INPUT);
  pinMode(led1Pin, OUTPUT);
  pinMode(led2Pin, OUTPUT);
  pinMode(led3Pin, OUTPUT);

  FlexiTimer2::set(200, UpdateService);
  FlexiTimer2::start();

  pinMode(12, INPUT_PULLUP);
  
  radio.begin();
  radio.openWritingPipe(address);
  radio.setPALevel(RF24_PA_MIN);
  radio.stopListening();

  Serial.println("------------------------------------------------------------------------------------------");
  Serial.println("Please start to operate the remote...");
}

void loop() {
  int pot1Value = analogRead(pot1Pin);
  int pot2Value = analogRead(pot2Pin);
  int joystickXValue = analogRead(joystickXPin);
  int joystickYValue = analogRead(joystickYPin);
  bool joystickZValue = digitalRead(joystickZPin);
  bool s1Value = digitalRead(s1Pin);
  bool s2Value = digitalRead(s2Pin);
  bool s3Value = digitalRead(s3Pin);

  static int pot1ValueBefore = pot1Value;
  static int pot2ValueBefore = pot2Value;
  static int joystickXValueBefore = joystickXValue;
  static int joystickYValueBefore = joystickYValue;
  static bool joystickZValueBefore = joystickZValue;
  static bool s1ValueBefore = s1Value;
  static bool s2ValueBefore = s2Value;
  static bool s3ValueBefore = s3Value;

  static InputPin inputPin = InputPin::None;
  static InputPin inputPinBefore = inputPin;

  static int printCounter = 0;
  const int maxPrintCount = 15;

  const int potIgnoredLength = 8;

  if(abs(pot1Value - pot1ValueBefore) > potIgnoredLength) {
    /*
    if(inputPin != InputPin::Pot1) {
      inputPin = InputPin::Pot1;
      Serial.println("");
      Serial.print("Pot1: ");
    }
    pot1ValueBefore = pot1Value;
    Serial.print(pot1Value);
    Serial.print(", ");
    printCounter++;
    */
    pot1ValueBefore = pot1Value;

    char text[16];
    snprintf(text, sizeof(text), "pot1Value:%d", pot1Value);
    radio.write(&text, sizeof(text));
    Serial.print("pot1: ");
    Serial.println(pot1Value);
    delay(50);
  }

  if(abs(pot2Value - pot2ValueBefore) > potIgnoredLength) {
    /*
    if(inputPin != InputPin::Pot2) {
      inputPin = InputPin::Pot2;
      Serial.println("");
      Serial.print("Pot2: ");
    }
    pot2ValueBefore = pot2Value;
    Serial.print(pot2Value);
    Serial.print(", ");
    printCounter++;
    */
    pot2ValueBefore = pot2Value;

    char text[16];
    snprintf(text, sizeof(text), "pot2Value:%d", pot2Value);
    radio.write(&text, sizeof(text));
    Serial.print("pot2: ");
    Serial.println(pot2Value);
    delay(50);
  }

  if(abs(joystickXValue - joystickXValueBefore) > potIgnoredLength) {
    /*
    if(inputPin != InputPin::JoystickX) {
      inputPin = InputPin::JoystickX;
      Serial.println("");
      Serial.print("JoystickX: ");
    }
    joystickXValueBefore = joystickXValue;
    Serial.print(joystickXValue);
    Serial.print(", ");
    printCounter++;
    */
    joystickXValueBefore = joystickXValue;

    char text[16];
    snprintf(text, sizeof(text), "joystickX:%d", joystickXValue);
    radio.write(&text, sizeof(text));

    Serial.print("x: ");
    Serial.println(joystickXValue);

    resetX = false;

    delay(50);
  } else {
    if (!resetX) {
      joystickYValueBefore = joystickYValue;

      char text[16];
      snprintf(text, sizeof(text), "joystickX:%d", 512);
      radio.write(&text, sizeof(text));

      Serial.println("x: 512");

      resetX = true;

      delay(50);
    }
  }

  if(abs(joystickYValue - joystickYValueBefore) > potIgnoredLength) {
    /*
    if(inputPin != InputPin::JoystickY) {
      inputPin = InputPin::JoystickY;
      Serial.println("");
      Serial.print("JoystickY: ");
    }
    joystickYValueBefore = joystickYValue;
    Serial.print(joystickYValue);
    Serial.print(", ");
    printCounter++;
    */
    joystickYValueBefore = joystickYValue;

    char text[16];
    snprintf(text, sizeof(text), "joystickY:%d", joystickYValue);
    radio.write(&text, sizeof(text));

    Serial.print("y: ");
    Serial.println(joystickYValue);

    resetY = false;

    delay(50);
  } else {
    if (!resetY) {
      joystickYValueBefore = joystickYValue;

      char text[16];
      snprintf(text, sizeof(text), "joystickY:%d", 512);
      radio.write(&text, sizeof(text));

      Serial.println("y: 512");

      resetY = true;

      delay(50);
    }
  }

  if(joystickZValue != joystickZValueBefore) {
    delay(10);
    /*
    if(joystickZValue != joystickZValueBefore) {
      if(inputPin != InputPin::JoystickZ) {
        inputPin = InputPin::JoystickZ;
        Serial.println("");
        Serial.print("JoystickZ: ");
      }
      joystickZValueBefore = joystickZValue;
      Serial.print(joystickZValue);
      Serial.print(", ");
      printCounter++;
    }
    */
    joystickZValueBefore = joystickZValue;

    char text[16];
    snprintf(text, sizeof(text), "joystickZ:%d", joystickZValue);
    radio.write(&text, sizeof(text));
    Serial.print("z: ");
    Serial.println(joystickZValue);
    delay(50);
  }

  if(s1Value != s1ValueBefore) {
    delay(10);
    /*
    if(s1Value != s1ValueBefore) {
      if(inputPin != InputPin::S1) {
        inputPin = InputPin::S1;
        Serial.println("");
        Serial.print("S1: ");
      }
      s1ValueBefore = s1Value;
      Serial.print(s1Value);
      Serial.print(", ");
      printCounter++;
    }
    */
    s1ValueBefore = s1Value;

    char text[16];
    snprintf(text, sizeof(text), "s1Value:%d", s1Value);
    radio.write(&text, sizeof(text));
    Serial.print("s1: ");
    Serial.println(s1Value);
    delay(50);
  }

  if(s2Value != s2ValueBefore) {
    delay(10);
    /*
    if(s2Value != s2ValueBefore) {
      if(inputPin != InputPin::S2) {
        inputPin = InputPin::S2;
        Serial.println("");
        Serial.print("S2: ");
      }
      s2ValueBefore = s2Value;
      Serial.print(s2Value);
      Serial.print(", ");
      printCounter++;
    }
    */
    s2ValueBefore = s2Value;

    char text[16];
    snprintf(text, sizeof(text), "s2Value:%d", s2Value);
    radio.write(&text, sizeof(text));
    Serial.print("s2: ");
    Serial.println(s2Value);
    delay(50);
  }

  if(s3Value != s3ValueBefore) {
    delay(10);
    /*
    if(s3Value != s3ValueBefore) {
      if(inputPin != InputPin::S3) {
        inputPin = InputPin::S3;
        Serial.println("");
        Serial.print("S3: ");
      }
      s3ValueBefore = s3Value;
      Serial.print(s3Value);
      Serial.print(", ");
      printCounter++;
    }
    */
    s3ValueBefore = s3Value;

    char text[16];
    snprintf(text, sizeof(text), "s3Value:%d", s3Value);
    radio.write(&text, sizeof(text));
    Serial.print("s3: ");
    Serial.println(s3Value);
    delay(50);
  }

  /*
  if(inputPin != inputPinBefore) {
    printCounter = 0;
  }
  inputPinBefore = inputPin;
  
  if(printCounter >= maxPrintCount) {
    Serial.println("");
    Serial.print("    ");
    printCounter = 0;
  }
  */

  analogWrite(led1Pin, map(analogRead(pot1Pin), 0, 1023, 0, 255));
  analogWrite(led2Pin, map(analogRead(pot2Pin), 0, 1023, 0, 255));
}


void UpdateService()
{
  sei();

  UpdateStateLED();
}

void UpdateStateLED()
{
  const static int stepLength = 2;
  const static int intervalSteps = 3;
  static int ledState = ::ledState;
  static int counter = 0;

  if (counter / stepLength < abs(ledState))
  {
    if (counter % stepLength == 0)
      SetStateLed(ledState > 0 ? HIGH : LOW);
    else if (counter % stepLength == stepLength / 2)
      SetStateLed(ledState > 0 ? LOW : HIGH);
  }

  counter++;

  if (counter / stepLength >= abs(ledState) + intervalSteps)
  {
    ledState = ::ledState;
    counter = 0;
  }
}

void SetStateLed(bool state)
{
  digitalWrite(led3Pin, state);
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Freenove Hexapod Robot Kit | Base materials and electronics | $126.99 | <a href="https://store.freenove.com/products/fnk0031"> Link </a> |
| Raspberry Pi 4 Model B | Compute platform that supports live image processing and provides more computation power than the Arduino alone | $89.29 | <a href="https://www.amazon.com/Raspberry-Pi-Model-2GB/dp/B09TTNPB4J"> Link </a> |
| Raspberry Pi OV5647 Camera Module | For receiving visual input | $22.99 | <a href="https://www.amazon.com/HiLetgo-OV5647-Camera-Module-Raspberry/dp/B01D1D0DJ0"> Link </a> |
| USB microphone | For receiving audio input | $22.99 | <a href="https://www.amazon.com/Microphone-MAONO-Omnidirectional-Microphone-Recording-Broadcasting/dp/B074BLM973"> Link </a> |
| USB speaker | The main response to anything the camera sees or microphone hears | $13.99 | <a href="https://www.amazon.com/HONKYOB-Speaker-Computer-Multimedia-Notebook/dp/B075M7FHM1"> Link </a> |
| LED display | Secondary response mechanism | $??? | <a href=""> Link </a> |

<!--
# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
-->
