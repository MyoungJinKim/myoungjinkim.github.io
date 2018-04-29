---
layout: single
title: 아두이노로 장애물 피하기 로봇 만들기
date: 2018-04-30
categories: 
- arduino
tags:
- 아두이노
- 장애물 피하기
---

안녕하세요! RC카 만들기에 이어 두 번째 프로젝트로 운전 중 나타나는 다양한 장애물들을 피하며 다닐 수 있는, 일명 "장애물 피하기 로봇"을 만들어보기로 결정하였습니다. 장애물을 피하는 로봇을 만들기 위해서는 장애물의 유무를 감지할 수 있는 부품이 필요한데, 저는 초음파 센서 HR-SC04를 이용하기로 결정하였습니다. 또한, 장애물을 피할 때 장애물이 없는 쪽으로 피하기 위해 초음파 센서를 서보 모터 SG-90에 연결하여 왼쪽과 오른쪽에 있는 장애물의 거리를 재는 기능도 추가하였습니다.

## 필요한 부품들
첫 번쨰 프로젝트였던 RC카 만들기의 차체를 거의 그대로 사용하였습니다. 기본 뼈대가 되주는 아크릴 판과 바퀴 등을 포함한 키트는 그대로 사용하였고 건전지 홀더, 볼 캐스터를 구입하였고 그 외의 부품들은 원래 가지고 있엇던 것들을 사용하였습니다.

부품 | 수량 | 가격 | 구입처
아두이노 우노 R3 호환 보드 | 1 | 6,160원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=71796)
L298N 모터 드라이버 | 1 | 2,640원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=1221)
아두이노 로봇 2WD 프레임 키트 | 1 | 6,600원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=329290)
3/4인치 Pololu 볼 캐스터 | 1 | 4,180원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?&goodsno=427399)
HC-SR04 초음파 센서 | 1 | 1,100원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=119)
SG90 서보 모터 | 1 | 1,650원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=71795)
6xAA 배터리 홀더 | 1 | 1,430원 | [메카솔루션](http://mechasolution.com/shop/goods/goods_view.php?goodsno=8521)

## 조립하기

![처음 시작하는 차체 사진]({{ site.url }}/images/IMG_2481.jpg)

### 볼 캐스터

이전 프로젝트에서 사용했던 축이 돌아가는 형식의 캐스터는 차가 전진할 때 따라 움직여 차가 똑바로 가는데 지장을 줘 이번에는 그러한 문제를 최소화하기 위해 구 모양의 캐스터를 사용하였습니다. 볼캐스터의 높이가 차체 높이다 조금 낮아서 볼 캐스터를 위한 마운트를 3 D프린터로 인쇄하여 사용하였습니다. 3D 프린트 모델은 아래 주소에서 받으실 수 있습니다.

[볼 캐스터 마운트 모델](https://www.thingiverse.com/thing:2558770)

![볼캐스터 세트 구성물]({{ site.url }}/images/IMG_2484.jpg)
![볼캐스터 조립 사진1]({{ site.url }}/images/IMG_2485.jpg)
![볼캐스터 조립 사진2]({{ site.url }}/images/IMG_2486.jpg)
![볼캐스터 조립 사진3]({{ site.url }}/images/IMG_2487.jpg)
![볼캐스터 조립 사진4]({{ site.url }}/images/IMG_2489.jpg)

### 건전지 홀더

이전 프로젝트에서는 AA건전지 4개가 들어가는 배터리 홀더를 사용했었는데, 키트에 포함된 모터가 9V까지 동작하기 때문에 AA 건전지 6개가 들어가는 배터리 홀더로 변경하였습니다. 테이프를 이용하여도 아크릴 판에 잘 부착되지 않는다는 점을 고려하여 액자부착 테이프를 사용하여 고정해주었습니다.

![배터리 조립 사진]({{ site.url }}/images/IMG_2569.jpg)
![배터리 조립 사진]({{ site.url }}/images/IMG_2491.jpg)

### 아두이노 마운트
이전 프로젝트에서 아우이노의 배럴잭과 USB 포트가 전선이 많이 연결된 차체 뒤쪽으로 장착되어 프로그램을 고치거나 전원을 연결하기 불편하였습니다. 그래서 아두이노 마운트를 새로 디자인하여 위치를 차체 앞쪽으로 옮기고 USB포트와 전원 공급 잭이 옆으로 나오도록 고정하였습니다.

![아두이노 조립 구성물]({{ site.url }}/images/IMG_2492.jpg)
![아두이노 조립 사진1]({{ site.url }}/images/IMG_2493.jpg)
![아두이노 조립 사진2]({{ site.url }}/images/IMG_2494.jpg)
![아두이노 조립 사진3]({{ site.url }}/images/IMG_2495.jpg)

### L298N 마운트
이번 프로젝트에서는 L298N과 모터와 연결을 쉽게 하고, 다른 센서들과 아두이노과 배선을 쉽게 하기 위해 L298N을 아크릴 판의 밑 부분으로 빼놓았습니다.

![L298N 조립 사진]({{ site.url }}/images/IMG_2496.jpg)
![L298N 조립 사진]({{ site.url }}/images/IMG_2497.jpg)
![L298N과 아두이노 부착]({{ site.url }}/images/IMG_2500.jpg)

### 초음파 센서
초음파 센서와 다른 서보모터의 연결을 쉽게 하기 위해 영화 "월리"의 주인공인 Wall-E 의 디자인을 참고하여 마운트를 출력하였습니다. Wall-E 머리 디자인은 다음 주소에서 받으실 수 있습니다.

[Wall-E 디자인](https://www.thingiverse.com/thing:2605324)

![초음파 센서 구성물]({{ site.url }}/images/IMG_2501.jpg)
![초음파 센서 조립 사진1]({{ site.url }}/images/IMG_2502.jpg)
![초음파 센서 조립 사진2]({{ site.url }}/images/IMG_2503.jpg)
![초음파 센서 조립 사진3]({{ site.url }}/images/IMG_2504.jpg)
![초음파 센서 조립 사진4]({{ site.url }}/images/IMG_2509.jpg)

### 서보 모터
아크릴 판의 면적이 너무 작아 서보 모터를 마땅히 부착할 곳이 없어 따로 마운트를 새로 디자인하여 사용하였습니다.

![서보 모터 조립 사진1]({{ site.url }}/images/IMG_2506.jpg)
![서보 모터 조립 사진2]({{ site.url }}/images/IMG_2507.jpg)
![서보 모터와 초음파 센서 부착]({{ site.url }}/images/IMG_2510.jpg)

![서보 모터 조립 사진3]({{ site.url }}/images/IMG_2512.jpg)
![서보 모터와 초음파 센서 부착]({{ site.url }}/images/IMG_2514.jpg)

## 회로

### 회로도

![회로도 그림]({{ site.url }}/images/sch-arduino-obstacle-avoiding-robot.jpg)

### L298N 연결
L298N의 동작과 원리에 대해서는 [앞 프로젝트의 2번째 포스트](https://myoungjinkim.github.io/arduino/arduino-rc-car-part2/)에 자세히 설명해놓았으니 참고하세요. 아래는 L298N과 아두이노, 그리고 기타 부품들과의 연결을 정리해놓은 표입니다.

아두이노 | L298N       | 기타 
5V     | 5V          |
       | 12V         | 6xAA 건전지 홀더 (+)극
GND    | GND         | 6xAA 건전지 홀더 (-)극
5번 핀  | ENA         |  
3번 핀  | IN1         |  
4번 핀  | IN2         |  
8번 핀  | IN3         |  
7번 핀  | IN4         |  
6번 핀  | ENB         |  
       | OUT1        |  모터 A
       | OUT2        |  모터 A
       | OUT3        |  모터 B
       | OUT4        |  모터 B

### 초음파 센서와 서보 모터 연결
서보모터의 pwm은 아두이노의 9번핀에 연결하였습니다. 그 외의 vcc는 5v에, ground는 GND에 연결하였습니다.

초음파 센서의 TRIG 핀은 아두이노의 10번에, ECHO 핀은 11번에 연결해주었습니다. 아래는 이를 정리한 표입니다.

아두이노 | 서보 모터 | 초음파 센서
5V     | VCC     | VCC
GND    | Ground  | GND
9번 핀  | PWM     |
10번 핀 |         | TRIG
11번 핀 |         | ECHO

### 완성된 차체 모습

![완성된 차체 모습1]({{ site.url }}/images/IMG_2518.jpg)
![완성된 차체 모습2]({{ site.url }}/images/IMG_2567.jpg)

## 코드

### 동작 설명
이 로봇은 20cm 안에 장애물이 들어오면 피하도록 만들었습니다.

- 장애물이 없으면 직진합니다.
- 장애물이 감지 되면 멈춰서 왼쪽, 앞쪽, 오른쪽 거리를 측정합니다.
- 왼쪽과 오른쪽의 장애물이 앞쪽보다 가깝다면 뒤로 돕니다.
- 왼쪽 장애물이 멀면 왼쪽으로, 오른쪽 장애물이 멀면 오른쪽으로 돕니다.

### 코드 리포지토리
장애물 피하기 코드는 [코드 리포지토리](https://github.com/MyoungJinKim/arduino-obstacle-avoiding-robot)
에서 보실 수 있습니다.

## 실행 영상

<iframe width="560" height="315" src="https://www.youtube.com/embed/DNOzhHHyNNI" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## 후기
동영상에서 알 수 있듯 로봇은 원하는 데로 잘 동작하였으나, 몇가지 오류가 발생하는 것 역시 볼 수 있었습니다. 너무 가까이 있는 장애물은 감지하지 못하였고, 장애물이 없는 곳에서도 회전을 하는 경우가 있으며, 로봇이 똑바로 직진하지 못하는 문제가 그 예입니다. 앞으로 이러한 문제들을 해결할 수 있는 프로젝트들을 꾸준히 진행해 나갈려고 합니다. 이것으로 두 번째 프로젝트였던 장애물 회피 로봇 프로젝트를 마치겠습니다!