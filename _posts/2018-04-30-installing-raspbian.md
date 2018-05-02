---
title:  "Installing Raspbian Headless"
date:   2018-04-30 18:40:00 +0900
tags:   raspberry-pi
---

새로 구입한 Raspberry Pi에 제일 먼저 해야 하는 일은 OS를 설치하는 것입니다. Raspberry Pi에는 여러 종류의 Linux와 심지어 Windows 10 IOT 버전도 설치할 후 있습니다. 그 중에서 Raspbian은 Raspberry Pi에서 공식적으로 릴리즈하는 OS 입니다.

이 글을 쓰는 현재, Raspbian은 RASPBIAN STRETCH WITH DESKTOP과 RASPBIAN STRETCH LITE 두 종류가 있습니다. RASPBIAN STRETCH LITE는 GUI를 지원하지 않으므로, 디스플레이 장치와 키보드/마우스가 없는 embedded device에서 사용하기 알맞습니다. 여기에서는 LITE를 설치하는 과정을 설명합니다.

## 필요 장비
* Raspberry Pi
* Micro SD card - OS만 설치하면 4GB만 있어도 되지만, voice assistant까지 설치하려면 최소 16GB 이상 용량의 Micro SD 카드를 준비합니다.

## SD card에 RASPBIAN 설치하기

다음 부분은 PC에서 실행합니다.

참고로, Raspberry Pi에 OS를 설치하는 공식 매뉴얼은 [https://www.raspberrypi.org/documentation/installation/installing-images/README.md](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) 에 있습니다. 글을 쓴 이후에 방법이 변경될 수도 있으니, 아래 방법이 잘 안될때는 공식 매뉴얼을 참고하세요.

### RASPBIAN LITE 다운로드
[https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/) 에서 Raspbian 이미지를 다운로드 받을 수 있습니다. RASPBIAN STRETCH LITE를 다운로드 받습니다. 그리고, 다운로드 받은 압축 파일을 풉니다.

### Etcher로 OS 설치
[https://etcher.io/](https://etcher.io/) 에서 SD card에 이미지를 구울 프로그램인 Etcher를 다운로드 받아 설치합니다. Etcher를 실행시키고, 위에서 압축 해제한 Raspbian 이미지 파일과 SD card가 들어있는 드라이브를 선택한 후, “Flash!”를 누르면 이미지가 SD card에 구워집니다. 시간은 SD card 용량에 따라 몇 분 정도 걸립니다.

### ssh 활성화

SD card에 RASPBIAN이 다 설치되면 boot 드라이브가 마운트 됩니다. (마운트 되지 않으면 SD card를 다시 끼워보세요.)
Boot 드라이브의 루트디렉토리에 ssh란 이름의 빈 파일을 만듭니다. 이렇게 하면 RASPBIAN이 부팅될 때 ssh 서버가 활성화 됩니다.

### WIFI 설정

이 부분은 Raspberry Pi와 Ethernet으로 연결하려는 사람은 할 필요 없습니다.
Boot 드라이브의 루트 디렉토리에 wpa_supplicant.conf 파일을 만들고, 아래 내용을 입력합니다.
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

network={
        ssid="YOUR_SSID"
        psk="YOUR_PASSWD"
}
```
YOUR_SSID와 YOUR_PASSWD는 각자의 AP 환경에 맞게 바꿉니다.
이렇게 하면 RASPBIAN이 부팅될 때 마다 설정된 AP로 WIFI 접속을 하게 됩니다.

### Raspberry Pi의 IP 알아내기

SD card를 Raspberry Pi에 끼우고 부팅을 시킵니다. 약 20초 정도 지나면 부팅이 완료되고, AP에 WIFI로 연결될 것입니다. 이제, Raspberry Pi의 IP address를 알아내야 합니다. 다음과 같이, AP에서 알아내거나, Ubuntu/Windows PC에서 알아낼 수 있습니다.

#### AP에서 알아내기

일부 AP에서는 DHCP로 할당한 IP address list를 볼 수 있습니다. MAC address가 B8:27:EB로 시작하는 것이 Raspberry Pi 입니다.

#### Ubuntu

Ubuntu에서는 아래 명령어로 Raspberry Pi의 IP 주소를 알아낼 수 있습니다.
```
$ sudo nmap -sP 172.30.1.0/24
...
Nmap scan report for 172.30.1.15
Host is up (0.34s latency).
MAC Address: B8:27:EB:38:8D:33 (Raspberry Pi Foundation)
...
```
또는
```
$ sudo arp-scan -l
...
172.30.1.15	b8:27:eb:38:8d:33	(Unknown)
...
```
MAC address가 B8:27:EB로 시작하는 것이 Raspberry Pi 입니다.

#### Windows

Windows에서는 IP scanner 프로그램을 설치해야 합니다. [Angry IP scanner](https://sourceforge.net/projects/ipscan/) 가 유명합니다.

### ssh로 접속

Raspberry Pi의 IP address를 알아냈으면, putty 등의 ssh 터미널로 접속합니다.
맨 처음 로그인 ID와 password는 다음과 같습니다.
* ID: pi
* Password: raspberry

## Raspberry Pi 초기 설정

여기 부터는 ssh로 접속한 터미널에서 실행합니다.

Ssh 접속한 후, Raspberry Pi의 초기 설정을 해 주어야 합니다. 이 작업은 맨 처음 한 번만 해 주면 됩니다.

아래와 같이, 설치된 패키지를 최신으로 업데이트 합니다.
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

그리고, 아래 명령으로 Raspberry Pi 초기 설정을 해 줍니다.

```
$ sudo raspi-config
```

다음 아이템은 필수로 설정해 주는 것이 좋습니다.

2 Network Options에서
* N1 Hostname
* N2 Wi-fi

4 Localisation Options에서
* I1 Change Locale - en_US.UTF-8 선택, ko_KR.UTF-8 선택. 나머지는 모두 선택 해제 합니다. Default locale은 C.UTF-8을 선택합니다.
* I2 Change Timezone에서 Seoul 선택.
* I3에서 키보드를 us로 선택.

5 Interface Options에서
* P2에서 ssh enable

설정을 마치고, 아래 명령으로 재부팅 합니다.
```
$ sudo reboot
```
약 30초 후 재부팅이 완료되면 ssh로 다시 로그인 하고 아래 명령으로 네트워크 설정 및 현재 시간을 확인한다.

```
$ iw dev wlan0 link
$ iwlist wlan0 channel
$ ifconfig
$ date
```

## WIFI Concurrent mode 설정

(experimental)

/etc/udev/rules.d/90-wireless.rules 파일을 만들고 아래 내용을 적습니다. 이렇게 하면 다음 번 부팅때 uap0 라는 가상 네트워크 인터페이스가 만들어집니다. 이 uap0를 AP mode의 인터페이스로 사용할 것입니다.

```
ACTION=="add", SUBSYSTEM=="ieee80211", KERNEL=="phy0", \
    RUN+="/sbin/iw phy %k interface add ap0 type __ap", \
    RUN+="/bin/ip link set ap0 address b8:27:eb:38:8d:34"
```

아래 명령으로 dnsmasq와 hostapd 패키지를 설치합니다.
```
$ sudo apt-get install dnsmasq hostapd
```

/etc/dnsmasq.conf 파일의 맨 뒤에 다음 라인 추가
```
interface=lo,ap0
no-dhcp-interface=lo,wlan0
bind-interfaces
server=8.8.8.8
domain-needed
bogus-priv
dhcp-range=192.168.10.50,192.168.10.150,12h
```
/etc/hostapd/hostapd.conf 파일을 생성하고, 아래 내용 추가. YourApNameHere와 YourPassPhraseHere는 알아서 수정.
```
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0
interface=ap0
driver=nl80211
ssid=YourApNameHere
hw_mode=g
channel=11
wmm_enabled=0
macaddr_acl=0
auth_algs=1
wpa=2
wpa_passphrase=YourPassPhraseHere
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
rsn_pairwise=CCMP
```

/etc/wpa_supplicant/wpa_supplicant.conf 파일은 아래와 비슷하게 이미 만들어져 있을 것임. 거기에 id_str을 추가하자.
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YourSSID1"
    psk="YourPassphrase1"
    id_str="AP1"
}

network={
    ssid="YourSSID2"
    psk="YourPassphrase2"
    id_str="AP2"
}
```

/etc/network/interfaces 파일을 아래와 같이 수정하여 ap0에 static IP address를 부여함.
```
# interfaces(5) file used by ifup(8) and ifdown(8)

# Please note that this file is written to be used with dhcpcd
# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

auto lo
auto ap0
auto wlan0
iface lo inet loopback

allow-hotplug ap0
iface ap0 inet static
    address 192.168.10.1
    netmask 255.255.255.0
    hostapd /etc/hostapd/hostapd.conf

allow-hotplug wlan0
iface wlan0 inet manual
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface AP1 inet dhcp
iface AP2 inet dhcp
```

이렇게 하고 재부팅하면 스마트폰에서 YourApNameHere 란 이름의 AP가 보일 것입니다. 위 과정 중에서 /etc/hostapd/hostapd.conf 파일에서 AP 이름을 명시해 주었죠. hostapd.conf 파일에서 명시한 암호를 입력하면 스마트폰이 RPi에 접속이 될 것입니다. 스마트폰에서 ssh app을 실행하여 pi@192.168.10.1로 접속하면 ssh 연결을 할 수 있습니다.

## 참고 사이트

* [https://www.raspberrypi.org/documentation/installation/installing-images/README.md](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)
* [https://albeec13.github.io/2017/09/26/raspberry-pi-zero-w-simultaneous-ap-and-managed-mode-wifi/](https://albeec13.github.io/2017/09/26/raspberry-pi-zero-w-simultaneous-ap-and-managed-mode-wifi/)
