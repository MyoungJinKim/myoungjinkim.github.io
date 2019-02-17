---
title: 아두이노 라인 트레이서 만들기 - 두 개의 디지털 센서를 이용한 실험
date: 2019-02-11
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

안녕하세요! 줄 따라가는 로봇 만들기 두 번째 시간입니다. 오늘은 지난 포스트에서 완성했던 차체에 전선을 연결하고, 두개의 센서의 디지털 신호를 이용해서 줄을 따라가는 실험을 실행해보고자 합니다.

## 전선 배선

차체 조립에 대한 정보는 아래 링크를 참고해주시기 바랍니다.

[아두이노 줄 따라가는 로봇 조립하기]({{ site.url }}//arduino/arduino-line-follower-assemble/)

전선 배선은 아래 표에 나와있는 내용대로 진행하였습니다.

아두이노 |    모터   | L298N | TCRT5000 (왼쪽) | TCRT5000 (오른쪽)  |   HC-06
VCC    |          | VCC   |    VCC         |    VCC           |
GND    |          | GND   |    GND         |    GND           |
4      |          | IN2   |                |                  |
5      |          | ENA   |                |                  |
6      |          | ENB   |                |                  |
7      |          | IN4   |                |                  |
8      |          | IN3   |                |                  |
9      |          | IN1   |                |                  |
10     |          |       |      D0        |                  |
11     |          |       |                |     D0           |
12     |          |       |                |                  |    RX
13     |          |       |                |                  |    TX
       | motor A  | OUT1  |                |                  |
       | motor A  | OUT2  |                |                  |
       | motor B  | OUT3  |                |                  |
       | motor B  | OUT4  |                |                  |


## 트랙 만들기

트랙은 밑의 링크에 나와있는 pdf 파일을 인쇄하여 만들었습니다. 차가 다녀도 손상되지 않도록 두꺼운 종이를 덧붙여 완성하였습니다.

[아두이노 줄 따라가는 로봇 트랙](https://www.parallax.com/sites/default/files/downloads/28136-S2-PrintableTracks.pdf)

![두꺼운 종이에 붙인 사진]({{ site.url }}/images/IMG_0736.jpg)
![그 두꺼운 종이를 자른 사진]({{ site.url }}/images/IMG_0737.jpg)
![다양한 완성된 트랙들]({{ site.url }}/images/IMG_0734.jpg)

## 실험

### 원리

이번 실험은 센서 두개의 길 인식 여부에 따라 자동차를 움직입니다. 우선, 길폭보다 더 넓게 두개의 센서를 배치하여 처음에는 두 센서 모두 길을 인식하지 않도록 설정합니다. 이 때 자동차는 두 센서 중 적어도 하나에 길이 인식될 때까지 전진합니다. 전진 하던 중 어느 쪽 센서에 길이 인식되었다는 것은 곧 길이 휘어졌다는 것을 의미합니다. 즉, 오른쪽 센서가 길을 인식했다면 오른쪽으로 길이 휘어졌다는 것을 의미하여 왼쪽 바퀴를 돌려 자동차를 오른쪽으로 돌려야 하고 반대로 왼쪽 센서가 길을 인식했다면 왼쪽으로 길이 휘어졌다는 것을 의미하기에 오른쪽 바퀴를 돌려 자동차를 왼쪽으로 돌려야 합니다.

### 코드

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
#define IR_R 11
#define IR_L 10

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
  pinMode(IR_R, INPUT);
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
    bool leftDetectLine = digitalRead(IR_L) == HIGH;
    bool rightDetectLine = digitalRead(IR_R) == HIGH;
    
    if (leftDetectLine) {
      drive(speed, 0);
    } else if (rightDetectLine) {
      drive(0, speed);
    } else {
      drive (speed, speed);
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

다양한 속도로 시험을 하기 위해 블루투스를 통해 명령을 받아 움직이도록 만들었습니다. 시리얼 모니터에 `g pwm값`을 주면 pwm 값으로 줄을 따라 이동합니다. 시리얼 모니터에 `s`를 입력하면 차가 멈춥니다.

원리에서 설명한 것과 같이 한 센서가 줄을 발견하면 그 방향의 바퀴를 멈추고 반대쪽 바퀴를 움직여 차의 방향을 조정하였습니다.

### 결과

<iframe width="560" height="315" src="https://www.youtube.com/embed/j-CHpuxHI_4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## 느낀 점

이번 실험에서 가장 아쉬웠던 점은 길이 차에 비해 작고 경사가 심해 차가 움직이는데 제약이 생긴다는 것이었습니다. 길이 좀만 더 크고 완만했다면 차를 돌리기 위해 한 쪽 바퀴를 멈추기 보다는 다른 쪽 바퀴에 비해 속도를 줄여 더욱 부드럽게 움직일 수 있을 것입니다. 반대로 이보다 더 길의 경사가 가파르면 한 바퀴를 아예 뒤로 돌려 더 급격하게 방향을 조정할 수 있습니다. 트랙에 따라 조정 방법이 달라진다는 것을 확인 하였습니다.
