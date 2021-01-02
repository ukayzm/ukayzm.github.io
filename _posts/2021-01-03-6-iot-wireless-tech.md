---
title:  IoT Wireless Technologies - IoT 무선 통신 기술
tags:   iot
header:
  teaser: /assets/img/posts/iot-3337536_1280.png
  image: /assets/img/posts/iot-3337536_1280.png
date:   2021-01-02 10:00:00 +0900
sidebar:
  - nav: docs
---

IoT 기기에는 반드시 통신 기능이 들어간다. IoT는 스펙트럼이 매우 넓은 개념이라서, 하나의 통신 기술로 모든 경우를 커버할 수는 없다. 여기에서는 IoT에서 많이 채용되는 6가지 무선 통신 기술을 소개하고 비교한다. 이 글은 BEHRTECH에 실린 [6 Leading Types of IoT Wireless Tech and Their Best Use Cases](https://behrtech.com/blog/6-leading-types-of-iot-wireless-tech-and-their-best-use-cases/) 를 많이 참조했다.

6개의 기술들은 각각 장단점이 있다. 그리고, 응용 분야 별로 필요한 요소도 서로 다르다. 그래서, 내가 만들고자 하는 유스케이스에 맞는 기술을 고르려면 전송 속도, QoS, 보안, 전력소모, 관리, 가격 등 여러 요소를 고려해야 한다. 데이터 전송 속도는 대체로 전력소모에 비례한다. 도달거리나 가격 등은 서로 다르다. 아래 그림은 각 기술들의 특징을 한 눈에 보여준다.

{:refdef: style="text-align: center;"}
![start](/assets/img/posts/iot-wireless-techs.jpg) 
{: refdef}

# LPWANs

LPWANs (Low Power Wide Area Networks)란, 하나의 기술이 아니라 전력소모가 적고 큰 규모의 네트워크에 적용되는 기술의 통칭이다. 적은 배터리로도 수 년동안 지속될 수 있을 정도로 전력소모가 적고, 산업용이나 캠퍼스처럼 장거리를 지원한다. Sigfox, LoRa, NB-IOT 등이 대표적인 LPWAN 기술이다.

LPWAN의 저전력이라서 전송 속도가 매우 낮기 때문에, 센서가 주력인 경우에 사용하기 좋다. 예를 들면 자원 추적, 환경 모니터링, 시설 관리, 점유 검출 등이다.

# Cellular (3G/4G/5G)

Cellular 네트워크는 모든 곳에서, 항상 가용하고, 보안도 좋다는 점에서 높은 신뢰성을 제공한다. 반면에 가격이 비싸고 소비전력도 높다.

배터리로 구동되는 센서에는 적합하지 않지만, 커넥티드 카(Connected Car) 또는 화물 추적 관리같은 곳에서 사용하기에 알맞다. 특히, 5G는 고속, 초저지연이기 때문에 실시간 비디오 감시, 커넥티드 의료, 그밖에 실시간이 필요한 산업에서 사용될 것으로 예상된다. 

# Zigbee

Zigbee는 단거리, 저전력이고 IEEE 802.15.4 표준안이다. 최대 100m 정도로 단거리 이기는 하지만, 여러 노드를 통해 데이터를 전달하는 mesh topology를 지원하기 때문에, 이를 이용하면 LPWAN에 비해서 저전력이면서도 높은 전송속도를 얻을 수 있다. 

이론적으로는 Wi-Fi나 LPWAN보다 좋긴 한데, 관리가 어렵다는 단점이 있다.

# Bluetooth and BLE

Bluetooth는 현재 Personal Area Network 시장에서 가장 많이 사용되는 기술이다. BLE는 Bluetooth Low Energy의 약자로, 소비전력이 적기 때문에 IoT에서 사용되기에 알맞다. 이미, 스마트폰, 전자제품, 주변기기뿐 아니라, 스마트 워치, 의료기기, 스마트홈 등에서 널리 사용되고 있다.

2017년에는 Bluetooth Mesh 스펙이 배포되어서, mesh network까지 확장될 수 있게 되었다. BLE beacon network은 실내에서 위치를 추적하는 기능을 제공한다.

# Wi-Fi

Wi-Fi는 설명이 불필요할 정도로 산업, 가정 할 것 없이 가장 많이 쓰이는 기술이다. 차세대 Wi-Fi 기술인 Wi-Fi 6는 최대 9.6Gbps의 전송속도를 지원한다. 스마트홈 기기처럼, 가정 내에서 전원을 연결해서 사용하는 경우에는 최고의 솔루션이다. 

하지만 IoT에서는 많이 사용되지 않는데, IoT 기기에서 필요한 것보다 너무 고사양이기 때문이다. 배터리로 구동되기에는 전력 소모가 많고, 도달 범위가 좁아서 넓은 네트워크를 구성할 수 없다. 

# RFID

RFID (Radio Frequency Identification)은 적은 양의 데이터를 아주 짧은 거리로 전송하는 기술이다. RFID tag가 사용하는 전력은 reader기에서 무선으로 전송되기 때문에, RFID tag에는 배터리가 필요 없다. 이것이 가장 큰 장점이다. RFID tag를 제품에 붙여, 소매나 배송 등의 분야에서 주로 사용된다. 

# Summary

아래 표는 주요 IoT 분야에 따라 알맞은 무선 통신 기술이다.

|   | LPWAN | Cellular | Zigbee | BLE | Wi-Fi | RFID 
|---|---|---|---|---|---|
| 산업용 IoT | ● | ◯ | ◯ |   |   |  
| 스마트 검침 | ● |   |   |   |   |  
| 스마트 도시 | ● |   |   |   |   |  
| 스마트 빌딩 | ● |   | ◯ | ◯ |   |  
| 스마트 홈  |   |   | ● | ● | ● |
| 웨어러블   | ◯ |   |   | ● |   |  
| 커넥티드 카 |  | ● |   |   | ◯ |   
| 원격 의료  |   | ● |   | ● |   |
| 스마트 소매 |   | ◯ |   | ● | ◯ | ● 
| 배송, 추적 | ◯ | ● |   |   |   | ●
| 스마트 농업 | ● |   |   |   |   |

● Highly applicable
◯ Moderately applicable
