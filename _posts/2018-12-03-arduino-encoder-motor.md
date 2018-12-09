---
layout: single
title: 아두이노 직각 위상 인코더(quadrature encoder) 모터 사용하기
date: 2018-12-02
categories: 
- arduino
tags:
- 아두이노
- 직각 위상 인코더
- quadrature encoder
- DC 모터
---

안녕하세요! 오늘은 DF 로봇사에서 생산한 직각 위상 인코더가 붙어있는 DC 모터의 사용법을 알아보고자 합니다. 이 모터를 살펴보게 된 것은 이전 포스트에서 사용하였던 인코더인 HC-020K에 몇가지 문제가 있었기 때문입니다. 이전 [비례 제어기 포스트]({{ site.url }}/arduino/arduino-propotional-drive/)에서 100ms마다 인코더를 이용하여 오류를 측정해 자세를 바로잡았습니다. 100ms 안에는 왼쪽과 오른쪽 바퀴의 차이가 1내지 2 tick 정도 밖에 나지 않았습니다. 그런데, 이전 [인코더 포스트]({{ site.url }}/arduino/arduino-hc-020k/)에서 볼 수 있듯이 케페시터로 인코더의 오류를 수정하였음에도 불구하고 여전히 1내지 2 tick 정도의 오류가 발생하였습니다. 이러한 오류는 100ms 단위의 측정 치명적이어서 자동차를 제대로 제어할 수가 없게 되었습니다. 이러한 상황을 개선하고자 높은 해상도로 정확한 측정값을 얻을 방법이 필요해져서 새로운 인코더를 찾아보다가 이 모터가 적절하여  사용하게 되었습니다.

## 사용한 부품

부품 | 가격 | 구입처
6V 마이크로 기어드 DC 모터 w/ SJ01 | 1,1000원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=329766&category=131003)

## 모터 스펙

- Gear ratio: 120:1
- No-load speed @ 6V: 160 rpm
- No-load speed @ 3V: 60 rpm
- No-load current @ 6V: 0.17A
- No-load current @ 3V: 0.14A
- Max Stall current: 2.8A
- Max Stall torque: 0.8kgf.cm
- Rated torque: 0.2kgf.cm
- Encoder operating voltage: 4.5~7.5V
- Motor operating voltage: 3~7.5V (Rated voltage 6V)
- Operating ambient temperature: -10~+60℃
- Weight: 50g

이 모터는 6 볼트에서 동작하는 DC 모터에 직각 위상 인코더가 포함되어있는 모터입니다. 기어 비율이 120:1이므로 모터 축이 120 바퀴 회전하면 바퀴가 한바퀴 회전 합니다. 모터 축에 위치한 디스크가 한 바퀴 돌면면 16펄스가 나오므로, 바퀴가 한 바퀴 돌 때 16x120 = 1920 펄스가 발생한다는 것을 알 수 있습니다.

## 모터 연결

![모터의 전선 연결]({{ site.url }}/images/FIT0450.png)

그림과 같이 위에 있는 1번 핀과 2번 핀은 모터 구동을 위한 전원 선이므로 L298N과 같은 모터드라이버에 연결합니다. 아래에 있는 3, 4, 5, 6번 핀은 직각 위상 인코더가 사용하는 핀으로 아두이노의 신호 처리 핀에 연결합니다. 3번은 인코더 A 핀의 츨력, 4번은 인코더 B핀의 출력, 5번은 그라운드, 6번은 VCC에 연결해 줍니다.

## 직각 위상 인코더 동작 원리

![직각 위상 인코더 원리 설명]({{ site.url }}/images/quadrature-encoder-pulse.png)

이 모터에서 특징적인 것은 앞으로 회전할 때와 뒤로 회전할 때가 구분 가능하다는 것입니다. 그림에서 볼 수 있듯 직각 위상 인코더에는 센서가 두 개 달려있으며, 회전할 때 두 센서에서 나온 신호들의 주기가 1/4만큼 떨어지도록 설계되어 있습니다. 모터가 시계방향으로 돌고 있을 때는 그림의 오른쪽에서 왼쪽으로 볼 때와 같이 A의 신호가 LOW애서 HIGH로 바뀔 때 B의 신호는 LOW를 나타냅니다. 시계 반대 방향으로 돌고 있을 때는 그림의 왼쪽에서 오른쪽으로 볼 때와 같이 A의 신호가 LOW에서 HIGH로 바뀔 때 B의 신호는 HIGH를 가르키고 있게 됩니다.

## 회로도

![회로도]({{ site.url }}/images/magnetic_motor.png)

## 코드

직각 위상 인코더의 출력을 확인해 보기 위해 아래와 같은 간단한 코드를 작성해 보았습니다. 이 코드는 아두이노의 외부 인터럽트를 사용하여 인코더 A핀의 출력이 변할 때마다 모터의 회전 방향에 따라 인코더 출력값을 증가하거나 감소하도록 설계되어 있습니다. 모터의 회전 방향을 알기 위해 A의 이전 출려과 현재 출력을 비교하여 LOW에서 HIGH로 변화했을 때, B의 값이 LOW인지 아니면 HIGH인지에 따라 방향을 결정합니다.

````
void ISR_encoderB() {
  byte encoderBA = digitalRead(ENCODER_B_A);
  if (encoderBLast == LOW && encoderBA == HIGH) {
    byte encoderBB = digitalRead(ENCODER_B_B);
    if (encoderBB == LOW) {
      encoderBDir = true;
    } else if (encoderBB == HIGH) {
      encoderBDir = false;
    }
  }
  encoderBLast = encoderBA;
  if (encoderBDir) {
    encoderBCount++;
  } else {
    encoderBCount--;
  }
}
````

전체 코드는 아래 gist에서 확인할 수 있습니다. 이 코드는 시리얼 모니터를 통해 'f' 명령을 보내면 시계방향으로, 'b' 명령을 보내면 반시계 방향으로 회전하고, 's' 명령을 보내면 정지하는 프로그램입니다.

<script src="https://gist.github.com/MyoungJinKim/a5b50816615f5f679a42a73603633ea8.js"></script>

## 실행 결과

![실험 결과]({{ site.url }}/images/quadrature-encoder-test.png)

