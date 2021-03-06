---
layout: single
title: 아두이노 초음파 센서 HC-SR04 조작하기
date: 2018-04-29
categories: 
- arduino
tags:
- 아두이노
- 초음파 센서
- HC-SR04
---

안녕하세요! 이번에도 서보 모터와 마찬가지로 두 번째 프로젝트를 위해 미리 초음파 센서에 대해 알아보도록 하겠습니다.

## 초음파 센서란?

초음파 센서는 전기 신호를 초음파로 바꿔 전송한 뒤에 돌아오는 초음파를 신호를 받아 처리하는 장치입니다. 용도에 따라 다양한 초음파 센서가 존재하는데요, 병원에서는 몸 안을 보기 위해 초음파 검사용 장비로 사용하기도 하고, 기계 내부를 들여다 보는 장치로 사용하기도 합니다. 우리가 살펴볼 HC-SR04는 초음파 신호가 돌아온 시간을 재서 물체와의 거리를 측정하는 용도로 사용하였습니다.
어떤 용도로 사용이 되든 모든 초음파 센서에는 초음파 신호를 보내는 장치와 돌아온 신호를 받아드리는 장치가 있습니다.

## HC-SR04 스펙

- 정격 전압: DC 5V
- 동작 전류: 15mA
- 동작 주기: 40Hz
- 최대 측정 거리: 4m
- 최소 측정 거리: 2cm
- 측정 가능 각도: 15도

5V에 15mA에서 작동하기 때문에 아두이노 핀에 직접 연결하여 사용할 수 있습니다. 또한, 동작 주기가 40Hz이므로 한 번 측정하고 25ms 정도를 쉬어줘야 합니다. 정면에서 15도 안에 있는 물체 까지의 거리를 잴 수 있는데, 2cm에서 4m 범위에 들어있지 않은 물체는 감지할 수 없습니다.

## 회로 연결 방법

Vcc는 아두이노의 5V에 연결하고, GND는 아두이노의 GND에 연결합니다. Trig와 Echo는 아무 디지털 핀에 연결해도 상관 없습니다.

## 동작 방법

![초음파 센서 동작 방법]({{ site.url }}/images/HC-SR04-timing.png)

측정을 원한다면 Trig 핀에 10us 동안 High 신호를 줍니다. Trig 핀이 Low로 바뀌면 센서에서 총 8번 초음파를 쏩니다. 다 쏘고 나면 Echo를 High로 만들고, 다시 초음파가 돌아오면 Low로 변경합니다.
이때, Echo 핀이 High 였던 시간을 재서 물체와의 거리를 측정합니다. 보통 소리가 초당 340m의 속도로 움직이고, 앞의 단계에서 측정한 시간은 물체까지 왕복한 거리로 물체 까지의 거리의 2배입니다. 즉, 우리는 시간을 거리로 환산한 후 그 값을 2로 나누어 거리를 구해야 합니다. 이 원리를 공식화 하여 물체까지의 거리를 x(cm)라 하고, High였던 시간을 t(us)라고 하여 비례식을 풀면 밑의 식이 성립하게 됩니다.

````
x : 34000 = t/2 : 1000000 
x = t * 0.017 또는 x = t / 58
````
다만, 온도와 습도에 따라 음속이 달라지기 떄문에 온도와 습도를 측정하면 더욱 정확한 값을 구할 수 있습니다.

아래의 코드는 Trig를 아두이노의 10번에, Echo를 11번에 연결하여 물체와의 거리를 측정하는 예입니다.

<script src="https://gist.github.com/MyoungJinKim/9828c03d458ba118c5f6af80d6be3b5b.js"></script>

## NewPing 라이브러리 사용

NewPing 라이브러리는 여러 계산을 직접 하지 않고도 쉽게 거리를 측정할 수 있는 라이브러리 입니다. 사용하기 위해서는 NewPing 라이브러리를 인클루드 해야 합니다.

````
#include <NewPing.h>
````

다음으로, NewPing 개채를 설정해야 합니다. 개채는 Trig 번호, Echo 번호, 그리고 최대 거리로 초기화 하는데, 이들 중 최대거리는 설정하지 않을 시 자동으로 2m로 설정됩니다.

````
NewPing sonar(10, 11);    // Trig는 10번, Echo는 11번, 최대 거리는 2m
````

거리를 cm 단위로 측정하고 싶으면 `ping_cm()`, inch 단위로 측정하고 싶으면 `ping_in()`, 시간만을 알고 싶으면 `ping()` 메소드를 이용합니다.

아래는 NewPing 라이브러리를 사용하여 거리를 cm 단위로 측정한 예입니다.

<script src="https://gist.github.com/MyoungJinKim/5e54f7a5d49e678894145f6680c666b1.js"></script>

밑의 사진은 위 예제로 측정한 거리를 4 세그먼트 디스플레이로 표시한 예입니다. 이 사진을 통해 센서를 이용하여 정확히 거리를 잴 수 있음을 알 수 있습니다.

![초음파 센서 동작 모습]({{ site.url }}/images/IMG_2421.jpg)