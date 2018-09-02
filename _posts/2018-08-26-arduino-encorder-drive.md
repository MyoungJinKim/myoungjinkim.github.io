---
title: 아두이노 인코더를 이용해 자동차 움직이기
date: 2018-8-26 
categories: 
- arduino
tags:
- 아두이노
- 인코더
- HC-020K
- 일정한 거리 가기
- 블루투스 2.0
- HC-06
---

저희가 차에 장착한 인코더는 활용할 시 자동차가 이동한 거리를 구할 수 있는데요, 때문에 이러한 점을 이용하여 이번 자동차를 움직이게 한 후, 얼마나 갔는지 거리를 구해보려 합니다.

그렇다면 인코더는 어떤 원리로 작동하는 지부터 알아보도록 합니다. 인코더는 흔히 ‘tick’라고 부르는 구멍이 20개씩 따로 따로 뚫어져 있는데요, 한쪽에서 신호를 보낼 때, 구멍이 앞에 있으면 신호가 반대편으로 가 전달되고, 막힌 쪽이 앞에 있으면 신호가 반대편으로 가지 못한다는 점을 이용하여 전달된 신호가 20번 측정될 때 1바퀴 돌았다고 알 수 있는 것입니다.

## 차체 및 회로 구성

이 프로젝트에서는 앞의 "장애물 피하기 로봇 만들기" 프로젝트에서 사용하였던 차체를 그대로 이용했습니다. 다만, 이번에는 불필요한 초음파 센서와 서보 모트를 제거했고, 그 대신 HC-06 블루투스와 HC-020K 인코더를 새로이 연결해주었습니다. 각각의 연결 방법은 아래 두 링크를 참고하여 주세요.

* [아두이노 HC-020K 사용하기](/arduino/arduino-hc-020k/)
* [아두이노 HC-06 사용하기](/arduino/arduino-hc-06/)

### L298N 연결
L298N의 동작과 원리에 대해서는 [앞 프로젝트의 2번째 포스트](https://myoungjinkim.github.io/arduino/arduino-rc-car-part2/)에 자세히 설명해놓았으니 참고하세요. 아래는 L298N과 아두이노, 그리고 기타 부품들과의 연결을 정리해놓은 표입니다.

아두이노 | L298N       | 기타 
5V     | 5V          |
       | 12V         | 6xAA 건전지 홀더 (+)극
GND    | GND         | 6xAA 건전지 홀더 (-)극
5번 핀  | ENA         |  
4번 핀  | IN1         |  
9번 핀  | IN2         |  
7번 핀  | IN3         |  
8번 핀  | IN4         |  
6번 핀  | ENB         |  
       | OUT1        |  모터 A
       | OUT2        |  모터 A
       | OUT3        |  모터 B
       | OUT4        |  모터 B

### HC-020K와 HC-06 연결

아두이노 | HC-202K A | HC-202K B | HC-06
5V     | 5V        | 5V        | VCC
GND    | GND       | GND       | GND
2번 핀  |           | OUT       |
3번 핀  | OUT       |           |
10번 핀 |           |           | RX
11번 핀 |           |           | TX

## 인코더로 거리를 재기

이러한 원리를 아는 상태라면 거리를 재는 방법은 상당히 쉬워지는데요, 측정된 신호 수를 알면 몇 바퀴를 돌았는지를 알아낼 수 있으니, 바퀴의 둘레를 구하고 다음 비례식을 풀면 됩니다.

````
(측정된 신호 수) : (이동한 거리) = 20 : (바퀴 하나의 둘레)
````

## 일정한 거리 보내기

그렇다면 자동차가 이동한 거리를 알 수 있게 되었으니 지정한 거리를 움직일 수 있도록 조정해보도록 합니다. 위의 경우와 정확히 반대의 과정을 겪으면 되는데요, 이동한 거리를 상수로 설정하고 측정되어야 하는 ‘tick’의 수를 구하는 것입니다. 

````
ticks = 이동할 거리 * 20 / 바퀴의 둘레
````

그 다음 아두이노에서 해당 tick수를 가고 나면 모터를 멈추게 하는 원리인데요, 이를 코드로 옮겨 보았습니다.

````
const float RESOLUTION = 20.0; // 인코더의 해상도
const float DIAMETER = 65.0f;  // 바퀴의 지름
const float CIRCUMFERENCE = DIAMETER * PI;

void move(int distance, int pwm = 150) {
  // 움직일 ticks
  int ticksToMove = (int)(distance * RESOLUTION / CIRCUMFERENCE);

  // 인코더 값 초기화
  noInterrupts();
  encoderA = 0;
  encoderB = 0;
  interrupts();
  
  // 인코더 값이 움직여야 하는 틱수보다 작은 동안 이동
  while (encoderA <= ticksToMove && encoderB <= ticksToMove) {
    drive(pwm, pwm);
  }

  // 다 움직이면 멈춤
  drive(0, 0);
}
````


## 무선으로 조정하기

2번에서 결과적으로 자동차를 원하는 거리만큼 보낼 수 있게 되었으나 유선으로 차를 연결하여 보냈기에 보낼 수 있는 거리가 상당히 한정적이었으며 코드를 수정하는 과정에서 불편함을 느끼게 되었습니다. 때문에, 해결책으로 무선으로 자동차와 연결하는 방법을 생각해냈습니다. 앞 포스팅에서도 자주 했듯이 linvor 블루투스를 pc와 연결해주었고, 결과적으로 차를 움직일 수 있는 거리도 굉장히 늘어났고 코드를 수정할 때의 불편함도 조금 덜 수 있었습니다. 다음은 블루투스와의 연결을 추가한 코드입니다.

먼저 소프트웨어 시리얼 라이브러리를 인클루드 하고 `mySerial`이라는 이름으로 소프트웨어 시리얼 개체를 만들어 주었습니다. 

````
#include <SoftwareSerial.h>

#define TX 10
#define RX 11

SoftwareSerial mySerial(RX, TX);
````

아두이노 우노에서는 소프트웨어 시리얼 라이브러리가 57600 baud 까지만 지원하기 때문에 `setup()`에서 속도를 57600으로 지정했습니다. 물론 블루투스 AT 명령으로 장치의 통신속도도 맞춰놨습니다.

````
void setup() {
  // ...
  mySerial.begin(57600);
  // ...
}
````

마지막으로 `loop()`에서 무선 통신으로 통해 명령을 하여 자동차를 제어할 수 있게 만들었습니다.
명령어의 형식은 한글자 명령어와 파라미터가 오는 형태로 구성했는데, 지금은 이동을 나타내는 `G`라는 명령어만 만들었습니다. 이 뒤에는 거리와 pwm값을 쓰도록 하였습니다.

 ````
 void loop() {
  if (mySerial.available()) {
    char command = mySerial.read();
    if (command == 'G' || command == 'g') {
      int distance = mySerial.parseInt();
      int pwm = mySerial.parseInt();
      move(distance, pwm);
    }
  }
}
````

## 코드

전체 코드는 [깃허브 레포지토리](https://github.com/MyoungJinKim/arduino-encoder-drive/)에서 보실 수 있습니다.

## 결론

<iframe width="560" height="315" src="https://www.youtube.com/embed/JyPPnWpBoHQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>




실험 결과 명령한대로 자동차가 움직이긴 했으나, 한쪽 방향으로 계속 휘어지는 현상이 나타났습니다. 휘어짐의 정도가 pwm 값에 따라 달라지는지 확인해보기 위해 pwm 값이 각각 100, 150, 200인 경우를 실험해보았습니다. 결과적으로 pwm  값이 크면 기울어짐의 정도 줄기 했으나 여전히 경로가 왼쪽으로 휘어지는 것을 볼 수 있었습니다. 다음 프로젝트에서는 차를 똑바로 가도록 만들어보려 합니다.