---
title:  Visitor Alarm Telegram - 텔레그램으로 실시간 방문자 확인
tags:   face-recognition
header:
  teaser: /assets/img/posts/vat_title.png
  image: /assets/img/posts/vat_title.png
date:   2020-09-18 10:00:00 +0900
sidebar:
  - nav: docs
---

여기에서는 비디오에서 인식된 얼굴을 스마트폰에 설치된 텔레그램으로 전송하는 방법을 설명합니다. 이 기능을 사용하면, 웹캠을 현관이나 복도 같은 주요 장소에 설치하고, 사람이 인식되면 사용자의 핸드폰으로 실시간 사진을 전송받을 수 있습니다. 얼굴을 인식하고 사람별로 분류하는 기능은 [unknown face classifier](/unknown-face-classifier/)를 이용했고, 여기에 텔레그램으로 전송하는 기능을 추가했습니다.

# Visitor Alarm Telegram의 기능 

Visitor Alarm Telegram은 다음과 같은 기능을 가집니다.

* 비디오에서 얼굴을 인식
* 인식된 얼굴을 사람 별로 분류. 모르는 사람의 얼굴이라도 비슷한 얼굴끼리 분류
* 처음 보는 사람이 나타나거나, 알고 있는 사람이 다시 나타나면 텔레그램으로 알림을 보냄
* 텔레그램으로 현재 상태, 실시간 화면 등을 모니터링
* 텔레그램으로 인식된 사람의 이름을 변경하고, 리스트를 조회

## 소스코드

소스코드는 [여기](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram)에서 받을 수 있으며, 총 3개의 파일로 되어 있습니다. `person_db.py`와 `face_classifier.py`는 [unknown face classifier](/unknown-face-classifier/)와 거의 동일하고, `visitor_alarm_telegram.py`와의 인터페이스를 추가했습니다.

| file | 기능 |
|------|------|
| face_classifier.py | 얼굴을 인식하고 사람 별로 분류 |
| person_db.py | 얼굴과 사람 class를 정의하고, 파일로 저장 |
| visitor_alarm_telegram.py | 텔레그램과 통신을 담당 |

# 텔레그램 봇

텔레그램은 사용자가 직접 [Bot](https://core.telegram.org/bots)을 만들 수 있도록 HTTP 기반의 [Telegram Bot API](https://core.telegram.org/bots/api)를 제공합니다. 이를 이용하면 텔레그램 안에서 대화하듯이 Bot에게 명령을 내리고 응답을 받아볼 수 있습니다.

Telegram Bot API는 여러 언어로 포팅이 되어 있는데, 여기에서는 [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 패키지를 이용하여 만들었습니다.

## 텔레그램 봇과 채팅방 만들기

먼저 사용자의 스마트폰에 텔레그램을 설치해야 합니다. 그리고, 텔레그램 앱에서 `텔레그램 봇`을 만들어야 합니다. 만드는 방법은 구글에서 "[텔레그램 봇 만들기](https://www.google.com/search?newwindow=1&hl=ko&q=%ED%85%94%EB%A0%88%EA%B7%B8%EB%9E%A8+%EB%B4%87+%EB%A7%8C%EB%93%A4%EA%B8%B0)"를 검색하면 많이 나오니, 검색 결과를 참고해서 만드세요.

우리가 사용할 것은 만들어진 텔레그램 봇의 `이름`과 `token` 입니다. 텔레그램 봇의 `이름`은 텔레그램 대화방을 만들 때 사용됩니다. `token`은 텔레그램 봇의 고유번호와 비슷한 것으로, `visitor_alarm_telegram.py`를 실행할 때 입력 값으로 사용됩니다.

스마트폰에서 텔레그램을 실행시키고, 위에서 만든 텔레그램 봇의 `이름`을 검색하여 채팅방을 만듭니다. 이제부터 이 채팅방에서 텔레그램 봇과 통신을 하게 됩니다. 아직 텔레그램 봇을 실행시키지 않았기 때문에, 지금은 이 채팅방에 무언가를 입력해도 아무 응답이 없을 것입니다. 

## 메시지 전달 구조

텔레그램 봇은 웹캠이 설치된 PC에서 실행됩니다. 채팅방에는 텔레그램 봇과 사용자가 들어가 있습니다. 사용자가 채팅방에 메시지를 올리면, 텔레그램 봇이 그 메시지를 받아 명령을 수행합니다. 텔레그램 봇이 수행한 결과를 메시지로 올리고, 사용자는 그 채팅방으로 전송된 메시지를 스마트폰에서 받아볼 수 있습니다.

# 텔레그램 봇 실행하기

이제 PC에서 텔레그램 봇을 실행해 봅시다.

## 필요 패키지 설치

python 3.x로 실행해야 하고, virtualenv로 환경을 분리시키기를 추천합니다. 사전에 다음과 같은 패키지가 설치되어 있어야 합니다.

```bash
(py3) $ pip install opencv-python --upgrade
(py3) $ pip install opencv-contrib-python --upgrade
(py3) $ pip install dlib --upgrade
(py3) $ pip install face_recognition --upgrade
(py3) $ pip install imutils --upgrade
(py3) $ pip install python-telegram-bot --upgrade
(py3) $ pip install humanize --upgrade
```

## 채팅방 검증하기

아래와 같이 `visitor_alarm_telegram.py`을 실행해 보세요. `--token` 파라미터는 반드시 들어가야 하고, 위 단계에서 여러분이 만든 token을 주어야 합니다. 문제 없이 실행되었다면 아래와 같은 화면이 나옵니다. 

```bash
$ python visitor_alarm_telegram.py --token '1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI'
Visitor Alarm Telegram is started.
* srcfile = 0 (webcam)
* resize_ratio = 1.0
* sbf (second between frame processed) = 0.5
* similarity threshold = 0.42
* appearance_interval = 10
press ^C to stop...
```

사용자와 텔레그램 봇과의 통신을 확인하기 위해서, 스마트폰의 채팅방에서 `/help`라고 입력해 보세요. 아래와 같이 헬프 화면이 나와야 합니다. 

{:refdef: style="text-align: center;"}
![help](/assets/img/posts/vat_help.png) 
{: refdef}

텔레그램 봇을 종료시키고 싶으면, 아래와 같이 PC의 터미널에서 `^C`를 누르면 됩니다. 텔레그램 서버와의 통신 절차 때문에, 수 초 정도 시간이 걸립니다.

```bash
^C2020-09-18 18:37:26,066 - Received signal 2 (SIGINT), stopping...
Visitor Alarm Telegram is finished.
```

## 실행 옵션

`visitor_alarm_telegram.py`에는 아래와 같은 옵션을 줄 수 있습니다. 

```bash
(py3) $ python visitor_alarm_telegram.py -h
usage: visitor_alarm_telegram.py [-h] --token TOKEN [--srcfile SRCFILE]
                                 [--threshold THRESHOLD] [--sbf SBF]
                                 [--resize-ratio RESIZE_RATIO]
                                 [--appearance-interval APPEARANCE_INTERVAL]

optional arguments:
  -h, --help            show this help message and exit
  --token TOKEN         Telegram Bot Token
  --srcfile SRCFILE     Video file to process. If not specified, web cam is
                        used.
  --threshold THRESHOLD
                        threshold of the similarity (default=0.42)
  --sbf SBF             second between frame processed (default=0.5)
  --resize-ratio RESIZE_RATIO
                        resize the frame to process (less time, less accuracy)
  --appearance-interval APPEARANCE_INTERVAL
                        alarm interval second between appearance (default=10)
```

* `--token` 텔레그램 봇의 토큰, 반드시 필요함
* `--srcfile` 이 파라미터를 주면 입력 비디오로 웹캠이 아니라 비디오 파일이 사용됩니다. 테스트나 디버깅 용도로 사용.
* `--threshold` 인식된 얼굴을 동일인으로 분류할 기준. ([unknown-face-classifier](/unknown-face-classifier/)) 참고.
* `--sbf` 처리할 비디오 프레임의 간격. PC의 성능이 딸리면 값을 늘려주세요.
* `--resize-ratio` 비디오 크기를 줄여서 처리하도록 할 수 있습니다. 처리 시간이 줄어드는 대신에 정확도는 떨어집니다. PC의 성능이 딸리면 값을 0.5 정도로 줄여주세요.
* `--appearance-interval` 어떤 사람의 얼굴이, 이 시간보다 길게 보이지 않다가 다시 인식되면, 재출현으로 판단합니다.

## 사용 가능한 텔레그램 명령어

채팅창에서 텔레그램 봇에게 내릴 수 있는 명령은 아래와 같습니다.

| Available Commands | Comments |
|--------------------|----------|
| /help | 명령어 목록 출력 |
| /settings | 현재의 설정 값 출력  |
| /start | face classifier를 구동하여 얼굴 인식과 분류를 시작 |
| /stop | face classifier를 종료 |
| /status | person DB와 face classifier의 현재 상태 출력 |
| /shot | 현재 처리하고 있는 비디오 화면을 출력 |
| /rename old_name new_name | 인식된 사람의 이름 변경 |
| /list | person DB에 있는 사람의 리스트와 사진을 출력 |

# Example Result

Visitor Alarm Telegram은 웹캠으로 모니터링하는 것이 목적입니다. 하지만, 기능을 테스트 하거나 디버깅을 하려면 비디오 파일을 이용하는 것이 편합니다. 여기에서도 비디오 파일을 이용한 테스트 결과를 보여드리겠습니다. 테스트에 사용한 영상은 유튜브에 올려진 영화 '극한직업'의 예고편 파일 입니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xM1CIQd_X4c" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

이 비디오를 다운 받아서 `~/Videos/extj.mp4`에 저장했고, 아래와 같이 `--srcfile` 파라미터를 주고 실행했습니다.

```bash
$ python visitor_alarm_telegram.py --srcfile ~/Videos/extj.mp4 --token '1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI'
```

## 시작하기

텔레그램에서 `/start` 를 입력하면, PC에서 아래와 같은 로그와 함께 face classifier가 시작되고 얼굴 인식을 시작합니다.

```
2020-09-17 16:42:35,022 - Face classifier is started.
* srcfile = /home/rostude/Videos/extj.mp4
* size = 1280x720
* resize_ratio = 1.0 -> 1280x720
* frame_rate = 24.000 f/s
* process every 12 frames
```

그리고, 텔레그램으로는 아래와 같은 응답 메시지가 전달됩니다. 

{:refdef: style="text-align: center;"}
![start](/assets/img/posts/vat_start.png) 
{: refdef}

## 처음 보는 사람으로 분류되면

Face classifier가 인식된 얼굴을 처음 보는 사람으로 분류하면, PC에서는 아래와 같은 로그가 출력됩니다.

```bash
2020-09-17 16:42:38,288 - person_01 appeared first
2020-09-17 16:42:42,224 - person_02 appeared first
```

그리고, 텔레그램으로는 "person_XX appeared first"라는 문구와 함께 그 사람의 얼굴이 전달됩니다. 

{:refdef: style="text-align: center;"}
![person_first](/assets/img/posts/vat_person_first.png) 
{: refdef}

## 이전에 출현했던 사람이 다시 나타나면

이전에 출현한 적이 있는 사람이 다시 출현한 경우에는, PC에서는 아래와 같은 로그가 출력됩니다.

```
2020-09-17 16:43:04,643 - person_02 appeared again in 22 seconds since 2020-09-17 16:42:40
2020-09-17 16:43:10,124 - person_01 appeared again in 29 seconds since 2020-09-17 16:42:39
```

그리고, 텔레그램으로 "person_XX appeared again"라는 문구와 마지막으로 출현했던 시간 정보와 함께 그 사람의 얼굴이 전달됩니다. 이 때, 왼쪽의 얼굴은 이전에 인식되었던 얼굴이고 오른쪽은 현재 인식된 얼굴 입니다.

{:refdef: style="text-align: center;"}
![person_again](/assets/img/posts/vat_person_again.png) 
{: refdef}

## 텔레그램 명령어 실행 예

* /status

텔레그램에서 `/status`를 입력하면 face classifier의 현재의 상황을 보여줍니다. 아래 예를 보니, 지금까지 1분 5초 동안 실행했고, 131장의 화면을 분석했고 10 명의 사람이 인식되었다는 것을 알 수 있네요.

{:refdef: style="text-align: center;"}
![status](/assets/img/posts/vat_status.png) 
{: refdef}

* /rename

인식된 사람의 이름은 `person_XX` 같은 이름을 가집니다. `/rename` 명령으로 다른 이름으로 바꿀 수 있습니다. 아래 예에서는 영화상의 이름으로 바꿔 주었습니다.

{:refdef: style="text-align: center;"}
![rename](/assets/img/posts/vat_rename.png) 
{: refdef}

* /list

`/list` 명령은 지금까지 인식된 모든 사람의 사진과 이름, 얼굴 수를 보여줍니다. 위에서 `/rename`으로 바꿔 준 이름이 적용되어 있는 것이 보이시죠?

{:refdef: style="text-align: center;"}
![list](/assets/img/posts/vat_list.png) 
{: refdef}

* /shot

`/shot`은 현재 처리 중인 비디오 화면을 보여줍니다. 아래 예는 테스트 영상의 경우이지만, 웹캠이라면 현재 화면을 실시간으로 볼 수 있습니다. 때때로 웹캠을 설치한 장소를 실시간으로 감시하는 용도로 유용하게 사용할 수 있습니다.

{:refdef: style="text-align: center;"}
![shot](/assets/img/posts/vat_shot.png) 
{: refdef}

* /settings

`/settings` 명령은 설정 값을 보여줍니다. 

{:refdef: style="text-align: center;"}
![settings](/assets/img/posts/vat_settings.png) 
{: refdef}

* /stop

`/stop` 명령은 현재 실행중인 face classifier를 중지합니다.

# Reference

* [Python Face Recognition](/python-face-recognition/)
* [Unknown Face Classifier](/unknown-face-classifier/)
* [Source Code - https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram)
* [Telegram Bot](https://core.telegram.org/bots)
* [Telegram Bot API](https://core.telegram.org/bots/api)
* [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 
