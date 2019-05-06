---
title: 아두이노 라인 트레이서 만들기 - pid를 이용한 떨림 제어
date: 2019-05-06
categories: 
- arduino
tags:
- 아두이노
- 줄 따라가기
- TCRT5000
- line follower
- 라인 트레이서
- line tracer
- pid
---

안녕하세요! 아두이노 라인 트레이서 만들기 네 번째이자 마지막 시간입니다. 저번 시간에 하나의 아날로그 센서를 이용한 라인 트레이서를 만든 이후 실험해보았는데요, 트렉을 따라 잘 움직이긴 하지만 매우 심하게 떨리는 모습이 관찰되었습니다. 이러한 현상을 완화시키기 위해 여러 방안을 고민하다가 PID 제어기를 사용하기로 결정하였습니다.

## PID 제어기

PID는 proportional, integral, differential의 약자로 비례, 적분, 미분 제어를 뜻합니다. 자동차를 운영하는 등의 활동을 할 때 물체를 흔들리지 않고 똑바로 보낼 수 있도록 사용합니다. 

비례 제어기는 자동차의 측정값이 기준값과 벗어난 값인 error의 크기에 비례하여 자동차의 바퀴를 회전시킵니다. 이때, error의 크기에 따라 얼마나 크게 회전시킬지를 설정하기 위해 Kp라는 비례 상수를 이용합니다.

적분 제어기는 초기부터 쌓여있었던 error를 제거하기 위해 사용합니다. 비례 제어기는 현재 쌓인 error에 대해서만 반응하기 때문에 처음에 누적되어있었던 error를 제거하는 역할은 할 수 없습니다. 이러한 문제를 보완하기 위해 사용하는 제어기가 바로 적분 제어기이며, 이 역시 누적된 값에 따라 얼마나 크게 바퀴를 회전시킬지를 설정하기 위해 Ki라는 비례 상수를 이용합니다.

마지막으로 미분 제어기는 자동차의 갑작스러운 변화를 막기 위해 사용합니다. 누적된 값에 따라 바퀴를 회전시키면 변화가 일어날 때 바퀴가 과도할 정도로 급격히 회전하는 경우가 생깁니다. 미분 제어기는 이러한 문제를 해결하기 위해 사용합니다. 급격한 변화가 일어날 때만 적절히 바퀴를 움직이는 것이죠. 이 역시 바퀴가 회전하는 정도를 제어하기 위해 Kd라는 비례 상수를 이용합니다.

이때, 비례 상수인 Kp, Ki, Kd는 모두 반복된 실험을 통해 가장 적절한 값을 찾아냈습니다.

## 실험

### 코드

지난 포스트에서 사용하였던 하나의 아날로그 센서로 움직이는 코드에 PID 제어기 코드를 추가하였습니다.

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
double errorSum = 0.0;
double errorCurr = 0.0;
double errorPrev = 0.0;
double errorDiff = 0.0;

float Kp = 0.0;
float Kd = 0.0;
float Ki = 0.0;

long lastTime = 0;

void loop() {
  // check command over BLE
  if (mySerial.available()) {
    char command = mySerial.read();
    if (command == 'g') {
      speed = mySerial.parseInt();
      Kp = mySerial.parseFloat();
      Ki = mySerial.parseFloat();
      Kd = mySerial.parseFloat();
      errorCurr = 0;
      errorPrev = 0;
      errorDiff = 0;
      errorSum = 0;
    }
    else if (command == 's') {
      speed = 0;
    }
  }

  // drive line following robot
  if (speed > 0) {
    long currTime = millis();

    int value = analogRead(IR_L);

    if (value > 100) {
      errorCurr = value - 320;
      errorSum = errorSum + errorCurr;
      errorDiff = errorCurr - errorPrev;
      
      int gain = Kp * errorCurr + Ki * errorSum + Kd * errorDiff; 
      
      int pwmA = speed + gain;
      int pwmB = speed - gain;
      
      drive(pwmA, pwmB);

      lastTime = currTime;
      errorPrev = errorCurr;
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

### 1차 실험 결과

<iframe width="560" height="315" src="https://www.youtube.com/embed/ucoI4NWhbHI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


<iframe width="560" height="315" src="https://www.youtube.com/embed/sTtV_ZPFUMM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

위 동영상은 pid 제어기 없이 차를 움직였을 때 차의 움직임, 아래 동영상은 pid 제어기를 사용한 후 차를 움직인 동영상입니다. 복잡한 트렉에서 차를 움직여보기 전에 pid가 효과가 있는 지를 보기 위해 간단한 코스에서 차를 움직여보았습니다. 실험 결과 하나의 아날로그 센서로 진행하였던 실험에 비해 훨씬 떨림도 덜하고 더 빠른 속도에서 차를 움직여도 차가 탈선하지 않는다는 사실을 발견하였습니다. 하지만, 여전히 150 이상의 pwm 값에서는 제대로 동작하지 못하기도 했습니다.

### 2차 실험 결과

<iframe width="560" height="315" src="https://www.youtube.com/embed/jCyeodhfCtU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


<iframe width="560" height="315" src="https://www.youtube.com/embed/OtIikYALnt4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

1차 실험 결과와 마찬가지로 위 동영상은 pid 제어 없이 차를 움직였을 때, 아래 동영상은 pid 제어기를 사용한 후 차를 움직였을 떄의 결과입니다. 완만한 트렉에서의 실험이 성공적으로 끝난 이후 더 복잡한 트렉에서 실험을 해보았습니다. 실험 결과 예상에 비해 그렇게 큰 변화는 나타나지 않았습니다. 

### 느낀 점

첫 번쨰 실험을 통해 적절한 비례 상수 값을 찾을 수만 있다면 PID 제어기가 차를 부드럽게 움직이는데 매우 효과적인 수단이란 것을 알게 되었습니다. 하지만, 적절한 상수 값을 찾기 위해 많은 반복 실험을 해야 했으며, 때문에 체계적으로 이 값을 찾는 방법이 필요하겠다고 생각했습니다. 또한, 두 번째 실험 결과로 볼 수 있 듯 굴곡이이 심한 경우와 같은 특정 환경에서는 그다지 PID 제어기가 효과를 발휘하지 못하였습니다. 이에 더욱 효과적으로 자동차를 제어할 수 있는 방안을 마련해야겠다는 생각이 들었습니다.