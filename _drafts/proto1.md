---
title:  "Proto 1"
tags:   prototype
feature-img: "assets/img/posts/white-diagram-paper-under-pliers-1178498.jpg"
thumbnail:   "assets/img/posts/white-diagram-paper-under-pliers-1178498.jpg"
date:   2020-05-10 12:00:00 +0900
---

가장 쉬운 타입의 로봇을 만들어 보겠습니다. 

구동부로는 두 개의 모터에 각각 바퀴를 달 것입니다. 볼캐스터를 하나 달아서 총 세 개의 바퀴로 몸체를 지지합니다. 로봇청소기와 동일한 형태로 가장 간단한 구현 입니다.

바퀴는 두 개의 DC 모터로 서로 독립적으로 구동되어 전진, 후진, 방향 전환을 할 수 있습니다. 모터에 내장된 인코더를 이용하여 모터의 속도를 측정하고 원하는 속도로 정밀하게 제어할 수 있습니다.

가속도 자이로 센서를 이용하여 로봇의 방향을 측정합니다. 측정된 테이터로 모터의 회전속도를 결정하여 로봇의 이동 속도와 방향을 정밀하게 조정합니다.

전원은 다들 집에 하나쯤 있는 보조배터리를 사용합니다. 

가정에 흔하게 있는 TV 리모콘으로 로봇을 조종하고, 안드로이드 핸드폰으로 더 자연스럽게 조종합니다.

1. 하드웨어
   fritzing 설계도
   부품리스트 (나사 포함)
   사진 첨부
   멀티미터
2. 하드웨어 조립 & 연결 테스트
   a. Internal LED, Serial port
   b. IR 신호 수신
   c. MPU6050
   d. Bluetooth
   e. 모터 PWM 핀
   f. 모터 방향 핀
3. 기초 모터 테스트
   a. 방향 (전진, 후진)
   b. 최대 RPM 측정
   c. PWM vs RPM 측정
   d. IR로 모터 조종
4. PID로 정밀한 속도 제어
5. MPU6050으로 정밀한 각도 제어

## Reference

* [OpenCV TensorFlow Object Detection API](https://github.com/opencv/opencv/wiki/TensorFlow-Object-Detection-API)

