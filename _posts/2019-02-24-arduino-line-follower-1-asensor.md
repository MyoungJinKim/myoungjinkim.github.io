---
title: 아두이노 라인 트레이서 만들기 - 한 개의 아날로그 센서를 이용한 실험
date: 2019-02-24
categories: 
- arduino
tags:
- 아두이노
- 줄 따라가기
- TCRT5000
- line follower
- 라인 트레이서
- line tracer
---

안녕하세요! 아두이노 라인 트레이서 만들기 세 번째 시간입니다. 이번 포스트에서는 한 개의 센서의 아날로그 측정값을 이용하여 줄을 따라가는 실험을 진행해보고자 합니다. 디지털 센서를 쓰는 것 보다는 여러가지 고민 사항이 많아질 것 같습니다.

## 전선 배선

차체 조립에 대한 정보는 아래 링크를 참고해주시기 바랍니다.

[아두이노 줄 따라가는 로봇 조립하기]({{ site.url }}//arduino/arduino-line-follower-assemble/)

한전선 배선은 아래 표에 나와있는 내용대로 진행하였습니다. 오른쪽 TCRT5000을 떼어내고, 왼쪽 TCRT5000을 아날로그 핀 A0에 연결하였습니다.

아두이노 |    모터   | L298N | TCRT5000 (왼쪽) |   HC-06
VCC    |          | VCC   |    VCC         |
GND    |          | GND   |    GND         |
4      |          | IN2   |                |
5      |          | ENA   |                |
6      |          | ENB   |                |
7      |          | IN4   |                |
8      |          | IN3   |                |
9      |          | IN1   |                |
12     |          |       |                |    RX
13     |          |       |                |    TX
A0     |          |       |    A0          |
       | motor A  | OUT1  |                |
       | motor A  | OUT2  |                |
       | motor B  | OUT3  |                |
       | motor B  | OUT4  |                |

## 트랙

트랙은 지난 포스트에서 사용하였던 pdf 파일을 인쇄한 트랙과 함께 예전에 구매하였던 mindstorm 키트에 들어있던 완만한 트랙을 사용하였습니다.

아래 링크는 다양한 경로를 제작하기 위해 인쇄하였던 pdf 파일입니다.


[아두이노 줄 따라가는 로봇 트랙](https://www.parallax.com/sites/default/files/downloads/28136-S2-PrintableTracks.pdf)

## 실험

### 원리

TCRT5000의 아날로그 측정값은 바닥의 빛의 반사량을 알려줍니다. 측정값의 범위는 40에서 600 사이이며 반사량이 많을 수록 작은 값이 출력됩니다. 즉, 바닥이 흰색에 가까울 수록 측정값은 작아지고 바닥이 까만색에 가까울 수록 측정값은 커지게 됩니다.

이 실험에서는 하나의 아날로그 센서만을 이용하기 때문에 센서가 줄의 경계를 쫓아가도록 코드를 구성할 것입니다. 줄의 경계에서는 까만색 줄과 하얀색 바탕이 모두 인식되기 때문에 센서가 40과 600 사이의 값을 출력하게 됩니다. 저는 가운데 값인 320을 따라가도록 코드를 만들었습니다.

### 코드

320 이라는 경계를 설정해준 후 그보다 큰 값이 출력되면 줄 쪽으로, 작은 값이 출력되면 바탕 쪽으로 기울어져 있다고 판단하였습니다. 현재 왼쪽 센서로 줄의 왼쪽 경계를 쫓아가고 있는 상황을 고려하여 줄 쪽으로 기울어졌다고 판단될 경우 오른쪽 바퀴를, 바탕쪽으로 기울여졌다고 판단될 경우 왼쪽 바퀴를 돌려 값을 조율하였습니다.

````
#include <SoftwareSerial.h>

// Motor pins
#define ENA 5 
#define ENB 6
#define IN1 4
#define IN2 9
#define IN3 7
#define IN4 8

// Drives two motors according to the given pwm values.
void drive(int pwmA, int pwmB);

// TCRT5000 sensors
#define IR_L A0

// HC06 pins
#define TX 12
#define RX 13

SoftwareSerial mySerial(RX, TX);

void setup() {
  Serial.begin(115200);
  mySerial.begin(57600);

  // set motor pins to ouput
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN1, OUTPUT); 
  pinMode(IN2, OUTPUT); 
  pinMode(IN3, OUTPUT); 
  pinMode(IN4, OUTPUT); 

  // set TCRT500 pins to input
  pinMode(IR_L, INPUT);
}

// speed of motors in pwm value
int speed = 0;

void loop() {
  // check command over BLE
  if (mySerial.available()) {
    char command = mySerial.read();
    if (command == 'g') {
      speed = mySerial.parseInt();
    }
    else if (command == 's') {
      speed = 0;
    }
  }

  // drive line following robot
  if (speed > 0) {
    int value = analogRead(IR_L);
    if (value > 320) {
      drive(speed, 0);
    } else {
      drive(0, speed);
    }
  }  else {
    drive(0, 0);
  }
}

/*
 * Drives two motors according to the given pwm values.
 * 
 * To avoid write several functions for each motor direction, I simply
 * drive motor forward if the given value is positive and backward if
 * negative. To stop a motor, just give 0.
 * 
 * Since every DC motors have differenct characteristics, I stop a 
 * motor if the absolute value of the given pwm value is less than 
 * MOTOR_MIN_PWM to drive the vehicle as strait as possible.
 * 
 * @param pwmA PWM value of motor A. An integer value between 
 *             -255 and 255.
 * @param pwmB PWM value of motor B. An integer value between 
 *             -255 and 255.
 */

void drive(int pwmA, int pwmB) {
  // MOTOR A direction
  if (pwmA > 0) {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);
  } else if (pwmA < 0) {
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
  } else {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
  }

  // MOTOR B direction
  if (pwmB > 0) {
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
  } else if (pwmB < 0) {
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  } else {
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  }

  // speed of motors
  analogWrite(ENA, abs(pwmA));
  analogWrite(ENB, abs(pwmB));
}

````

### 실험 결과

<iframe width="560" height="315" src="https://www.youtube.com/embed/ucoI4NWhbHI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

실험 결과 완만한 길은 잘 쫓아가지만 급격한 경사가 등장하는 경우 줄을 따라가지 못하고 큰 원을 그리는 결과가 나타났습니다.

### 보완된 코드

위와 같은 상황을 해결하기 위해 아예 너무 하얀 쪽에 노출 될 경우 (이 코드의 경우 100보다 작을 때)는 두 바퀴를 서로 반대 방향으로 돌려 더욱 급하게 차체가 회전하도록 하였습니다.

````
#include <SoftwareSerial.h>

// Motor pins
#define ENA 5 
#define ENB 6
#define IN1 4
#define IN2 9
#define IN3 7
#define IN4 8

// Drives two motors according to the given pwm values.
void drive(int pwmA, int pwmB);

// TCRT5000 sensors
#define IR_L A0

// HC06 pins
#define TX 12
#define RX 13

SoftwareSerial mySerial(RX, TX);

void setup() {
  Serial.begin(115200);
  mySerial.begin(57600);

  // set motor pins to ouput
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN1, OUTPUT); 
  pinMode(IN2, OUTPUT); 
  pinMode(IN3, OUTPUT); 
  pinMode(IN4, OUTPUT); 

  // set TCRT500 pins to input
  pinMode(IR_L, INPUT);
}

// speed of motors in pwm value
int speed = 0;

void loop() {
  // check command over BLE
  if (mySerial.available()) {
    char command = mySerial.read();
    if (command == 'g') {
      speed = mySerial.parseInt();
    }
    else if (command == 's') {
      speed = 0;
    }
  }

  // drive line following robot
  if (speed > 0) {
    int value = analogRead(IR_L);
    if (value > 320) {
      drive(speed, 0);
    } else if (value > 100) {
      drive(0, speed);
    } else {
      drive(-speed, speed);
    }
   
  }  else {
    drive(0, 0);
  }
}

/*
 * Drives two motors according to the given pwm values.
 * 
 * To avoid write several functions for each motor direction, I simply
 * drive motor forward if the given value is positive and backward if
 * negative. To stop a motor, just give 0.
 * 
 * Since every DC motors have differenct characteristics, I stop a 
 * motor if the absolute value of the given pwm value is less than 
 * MOTOR_MIN_PWM to drive the vehicle as strait as possible.
 * 
 * @param pwmA PWM value of motor A. An integer value between 
 *             -255 and 255.
 * @param pwmB PWM value of motor B. An integer value between 
 *             -255 and 255.
 */

void drive(int pwmA, int pwmB) {
  // MOTOR A direction
  if (pwmA > 0) {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);
  } else if (pwmA < 0) {
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
  } else {
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);
  }

  // MOTOR B direction
  if (pwmB > 0) {
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
  } else if (pwmB < 0) {
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
  } else {
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
  }

  // speed of motors
  analogWrite(ENA, abs(pwmA));
  analogWrite(ENB, abs(pwmB));
}
````

### 실험 결과

<iframe width="560" height="315" src="https://www.youtube.com/embed/jCyeodhfCtU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

실험 결과에서 볼 수 있듯이 조금 급한 곡선도 잘 따라가는 것을 볼 수 있습니다. 그런데, 좌우로 일정한 속도로 움직이다보니 자체가 심하게 흔들렸습니다. 다음에는 이 부분을 보완해 보고자 합니다.