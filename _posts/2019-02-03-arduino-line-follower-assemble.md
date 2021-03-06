---
title: 아두이노 라인 트레이서 만들기 - 로봇 조립하기
date: 2019-02-03
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

안녕하세요! 이번에는 새로운 장기 프로젝트를 시작해 보려고 합니다. 바로 '줄 따라가는 로봇 만들기'인데요, 바닥에 그어진 줄을 감지하여 따라갈 수 있는 로봇을 여러가지 방식으로 만들어보고자 합니다. 이번 포스트는 그 첫 번째 시간으로 이후 다른 여러가지 로봇을 만드는데 뼈대가 되는 기본형 로봇을 만들 것 입니다.

## 차체 구성

이전 '장애물 피하기 로봇 만들기' 프로젝트에서 사용했던 차체 프레임을 사용하였습니다. 다만, 초음파 센서 등 이번 프로젝트에 필요없는 부품들을 제거하였고 TCRT5000을 추가하였습니다. 또한, 이전 프로젝트에서 알았냈던 인코더의 결함을 해결하기 위해 모터를 마이크로 기어드 DC 모터로 교체하였습니다.

TCRT5000 또는 마이크로 기어드 DC 모터에 대한 더 자세한 정보는 다음 링크를 참고해주시기 바랍니다.

부품 | 포스트
TCRT5000 | [아두이노 라인 트레이서 TCRT5000 사용하기]({{ site.url }}//arduino/arduino-TCRT5000/)
마이크로 기어드 DC 모터 | [아두이노 직각 위상 인코더(quadrature encoder) 모터 사용하기]({{ site.url }}//arduino/arduino-encoder-motor/)

## 차체 조립

![처음 시작하는 차체 사진]({{ site.url }}/images/IMG_0714.jpg)
![처음 시작하는 차체 사진]({{ site.url }}/images/IMG_0715.jpg)

사진에서 볼 수 있듯, 기본 차체 뼈대에 아두이노, 브레드 보드, 16V 건전지, L298N 모터 드라이버, 볼캐스터 등을 붙인 상태에서 조립을 시작하였습니다.

### 마이크로 기어드 DC 모터

인코더의 결함을 해결하기 위해 기존 DC 모터를 기어드 모터로 교체하였습니다. 3D 프린팅을 사용하여 모터 지지대를 인쇄하였으며, 모터에 전선을 연결한 이후 지지대에 나사로 고정하였습니다.

![기어드 DC 모터]({{ site.url }}/images/IMG_0720.jpg)
![기어드 DC 모터]({{ site.url }}/images/IMG_0721.jpg)
![기어드 DC 모터]({{ site.url }}/images/IMG_0724.jpg)
![기어드 DC 모터]({{ site.url }}/images/IMG_0725.jpg)

### 9V 건전지 

이전의 프로젝트들과 달리 이번 프로젝트에서는 아두이노 전원 공급용 9V 건전지를 차체 위에 설치하였습니다. 이는 DC 모터에 비해 기어드 모터가 차지하는 부피가 커서 공간이 부족해졌기 때문입니다.

![9V 건전지]({{ site.url }}/images/IMG_0726.jpg)

### TCRT5000

 이번 '줄 따라가기 로봇' 프로젝트에서 가장 중요한 역할을 하는 줄 감지 센서입니다. 모터와 마찬가지로 3D 프린팅을 사용하여 지지대를 인쇄하였으며, 감지를 하는데 최적거리가 2.5mm 임을 고려하여 센서의 끝이 바닥에서 2.4mm 떨어지도록 설계하였습니다. 지지대에 센서를 부착한 이후 나사를 이용하여 차체에 붙였습니다.

![TCRT5000]({{ site.url }}/images/IMG_0728.jpg)
![TCRT5000]({{ site.url }}/images/IMG_0729.jpg)
![TCRT5000]({{ site.url }}/images/IMG_0731.jpg)

## 완성된 차체

![완성된 차체]({{ site.url }}/images/IMG_0732.jpg)

이로써 '줄 따라가기 로봇' 프로젝트의 차체 조립을 마쳤습니다. 다음에는 전선 배치 및 실험 진행 후 결과에 대한 포스트를 하도록 하겠습니다.