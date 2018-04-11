---
title:  "2 Wheel Balancing Robot - Cases"
date:   2018-04-06 21:40:00 +0900
tags:   2-wheel-balancing-robot
---

인터넷이나 유튜브를 보면 많은 2 wheel balancing robot 작품을 접할 수 있습니다. 어떤 작품은 간신히 균형잡고 서있기 바쁜 것도 있지만, 놀랄만큼 재빠르고 날렵한 움직임을 보여주는 것도 있습니다. 심지어 넘어졌다가 스스로 일어나는 로봇도 있습니다.

모두 아두이노 또는 그와 비슷한 마이크로 콘트롤러를 이용하여 구현한 것들 입니다. 많은 사이트가 사용한 부품과 소스 코드를 공개하고 있습니다.

인상적이거나 시간을 두고 연구 분석할만한 사이트를 아래에 모아놓았습니다. 꼭 방문해 보시고, 사이트에 같이 올려진 동영상도 보시길 강력 추천 합니다.


## [http://www.brokking.net/yabr_main.html][brokking-yabr]

<iframe width="560" height="315" src="https://www.youtube.com/embed/6WWqo-Yr8lA" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* 1 x Arduino pro mini clone
* 1 x FTDI USB to TTL programmer for the Arduino pro mini
* 1 x Arduino Uno clone
* 1 x MPU-6050 gyro and accelerometer
* 2 x 2.4G wireless serial transceiver module
* 2 x 35mm Stepper motor
* 2 x Geeetech StepStick DRV8825
* 1 x Wired nunchuck controller for Wii
* 1 x Mini DC 7~28V to DC 5V step-down converter
* 1 x 11.1V 2200mAh 30C Li-polymer Battery
* 1 x B3AC 2S/3S Lipo balance charger

NEMA 14 스텝 모터 사용. 이곳을 참조하여 NEMA 17 모터로 만든 사람도 있음.
로봇 구동용과 리모콘 용도로 2개의 아두이노 사용
리모콘용 아두이노는 닌텐도 위 눈처크로부터 조이스틱 입력을 받아서 속도와 방향을 로봇 구동용 아두이노에게 블루투스로 전송함.
제작 과정을 유튜브 비디오로 볼 수 있음
움직임은 느린 편

## [Instructable 예제][instructable1]

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZYLb11xUsSY" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* DC 모터와 인코더 (모터 스펙은 나와있지 않음)
* L298N 모터 드라이버
* 2개의 18650 리튬이온 배터리, LM2596 DC/DC 컨버터
* 움직임은 느린 편


## [http://axelsdiy.brinkeby.se/?page_id=1447](http://axelsdiy.brinkeby.se/?page_id=1447)

<iframe width="560" height="315" src="https://www.youtube.com/embed/nT1FqvNbThU" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* Arduino Pro Micro (clone)
* 2 pcs. NEMA 14 stepper motors (1.8 degrees per step)
* 2 pcs. model airplane wheels, the hubs are drilled up from 4 mm to 5mm and press-fitted onto the motor shafts
* 2 pcs. A4988 Stepper motor driver modules
* MPU-6050 gyro/accelerometer breakout board
* 3 cell LiPo battery
* A buzzer, an LED, and a few resistors. (check the schematic in the downloadable folder)
* A few pin headers and sockets, some wire, and some heat shrink tubing
* About 20 pcs. M3 16 mm screws with nuts are used to assemble the robot
* Two toggle switches used for auxiliary functions

중간정도 속도에 안정적인 움직임을 보이고, 중심을 잘 잡음.
TMC2100 스텝모터 드라이버를 사용하도록 수정한 후, 엄청 조용해 졌다고 함.

PID에 대한 직관적인 설명이 인상적임:
![PID-theory](http://axelsdiy.brinkeby.se/wp-content/uploads/2015/11/balancingSystem700.png)


## [Make: 에 실린 예제](https://makezine.com/projects/arduroller-self-balancing-robot/)

<iframe width="560" height="315" src="https://www.youtube.com/embed/uXVwx9Gxp_E" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* DC 모터와 인코더
* Pololu 25D 34:1 motor with encoder (170 RPM, 50 oz-in (3.5 kg-cm), LP 6V, 2.4A)
* 큰 바퀴
* 모터 드라이버: SparkFun Ardumoto 쉴드
* L298, 2A per channel
* Battery, LiPo, 3S 11.1V
* Ardupilot APM 2.5 MCU

3D 프린터로 샤시 제작
중간 정도의 움직임


## [FK Engineering's Blog](http://fkeng.blogspot.kr/2016/04/self-balancing-two-wheels-mobile-robot.html?m=1)

<iframe width="560" height="315" src="https://www.youtube.com/embed/R6i-xSrRCec" allow="autoplay; encrypted-media" allowfullscreen></iframe>


## [Lukasz Bien의 작품](https://youtu.be/EwrQEsFmL4E )

<iframe width="560" height="315" src="https://www.youtube.com/embed/EwrQEsFmL4E" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* DC 모터와 인코더
* Pololu 37D 19:1 motor with encoder (500 RPM, 84 oz-in, 12V, 5A)
* MCU로 Teensy 3.2 사용
* 라즈베리파이로 카메라 구동
* BTS 7960 모터 드라이버 (43A)
* 18650 리튬이온 배터리
* 빠르고 안정적인 움직임


## [B-robot EVO](https://www.thingiverse.com/thing:2306541)

<iframe width="560" height="315" src="https://www.youtube.com/embed/d6J0ijMG3jI" allow="autoplay; encrypted-media" allowfullscreen></iframe>

[https://www.thingiverse.com/thing:1069256](https://www.thingiverse.com/thing:1069256)
[https://www.jjrobots.com/projects-2/b-robot/](https://www.jjrobots.com/projects-2/b-robot/)
[https://github.com/JJulio/b-robot](https://github.com/JJulio/b-robot)
[https://youtu.be/Rm9EDknlXHk](https://youtu.be/Rm9EDknlXHk)

* 스텝 모터 사용
* WIFI로 스마트폰과 연결하여 조종
* 명쾌하고 직관적인 PID 구조 설명
* 날렵하고 재빠른 움직임
* 팔을 이용하여 넘어져도 스스로 일어남
* 3D 프린터를 사용하여 프레임을 제작했음 (Thingiverse에 공개한 듯)
* 사용한 부품에 대한 설명은 없음


## [Zippy the Balancing Robot](https://github.com/elkayem/ZippyTheBalancingRobot)

<iframe width="560" height="315" src="https://www.youtube.com/embed/0io5SwitLzY" allow="autoplay; encrypted-media" allowfullscreen></iframe>

[https://youtu.be/V53LkU0RIlw](https://youtu.be/V53LkU0RIlw)
[https://youtu.be/0io5SwitLzY](https://youtu.be/0io5SwitLzY)

* 2 x VNH5180A 모터 드라이버 (\$3.72 each as SMD parts, or $8.69 each as evaluation boards from Mouser), 8A?
* DC 모터와 인코더
* Pololu 37D 30:1 motor with encoder (350 RPM, 110 oz-in, 12V, 5A) ($39.95 each from Pololu)
* 아두이노 나노 사용
* 3D 프린터로 프레임 제작
* 빠르고 부드러운 움직임
* ZIPPY Flightmax 3000mAh 3S 20C LiPo battery


## [BalancingWii 2.0](https://github.com/mahowik/BalancingWii)

<iframe width="560" height="315" src="https://www.youtube.com/embed/038e2j9nE3M" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* NEMA 17 스텝 모터
* A4988 모터 드라이버
* RC 리모콘으로 조종
* 모터의 힘만으로 스스로 일어남
* 빠르고 부드러운 움직임


## [Pololu Balboa 32U4 Balancing Robot](https://www.pololu.com/category/210/balboa-robot-and-accessories)

<iframe width="560" height="315" src="https://www.youtube.com/embed/sRtsc3EXL8A" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* [how to make](https://www.pololu.com/blog/662/how-to-make-a-balboa-robot-balance-part-1-selecting-mechanical-parts)
* Pololu사에서 제작한 키트
* DC 모터와 인코더
* 최소한의 부품, 가벼운 무게
* 아주 재빠른 움직임
* 모터의 힘만으로 스스로 일어남


[brokking-yabr]: http://www.brokking.net/yabr_main.html
[instructable1]: http://www.instructables.com/id/2-Wheel-Self-Balancing-Robot-by-using-Arduino-and-/
