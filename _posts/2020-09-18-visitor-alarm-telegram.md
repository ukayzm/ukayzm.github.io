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

이 포스트에서는 [unknown face classifier](/unknown-face-classifier/)를 확장하여, 인식된 얼굴을 스마트폰에 설치된 텔레그램으로 전송하는 기능을 설명합니다. `Unknown face classifier`를 안읽어 보신 분은 먼저 읽어 보시기를 강력히 권고합니다. 

소스코드는 [여기](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram)에서 받을 수 있습니다.

# 기능 

`Visitor Alarm Telegram`는 다음과 같은 기능을 가집니다.

* 비디오에서 얼굴을 인식 [python-face-recognition](/python-face-recognition/)
* 인식된 얼굴을 사람 별로 분류. 모르는 사람의 얼굴이라도 비슷한 얼굴끼리 분류 [unknown-face-classifier](/unknown-face-classifier/)
* 처음 보는 사람이 나타나거나, 알고 있는 사람이 다시 나타나면 텔레그램으로 알림을 보냄
* 사용자는 자신의 스마트폰에 설치된 텔레그램 앱으로 방문자 알림을 받을 수 있음

# 텔레그램 봇

텔레그램은 사용자가 직접 [Bot](https://core.telegram.org/bots)을 만들 수 있도록 HTTP 기반의 [Telegram Bot API](https://core.telegram.org/bots/api)를 제공합니다. 이를 이용하면 텔레그램 안에서 대화하듯이 Bot에게 명령을 내리고 응답을 받아볼 수 있습니다.

Visitor Alarm Telegram은 [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 패키지를 이용하여 만들었습니다.

## 텔레그램 봇 만들기

먼저 `텔레그램 봇`을 만들어야 합니다. 구글에서 ["텔레그램 봇 만들기](https://www.google.com/search?newwindow=1&hl=ko&q=%ED%85%94%EB%A0%88%EA%B7%B8%EB%9E%A8+%EB%B4%87+%EB%A7%8C%EB%93%A4%EA%B8%B0)를 검색하면 `텔레그램 봇`을 만드는 방법에 대해 자세한 설명이 많이 나오기 때문에, 여기에서 따로 설명을 하지는 않겠습니다.

우리가 필요한 것은 텔레그램 봇의 `이름`과 `token` 입니다. 봇의 `이름`은 텔레그램 대화방을 만들 때 사용됩니다. `token`은 `visitor_alarm_telegram.py`를 실행할 때 입력 값으로 사용됩니다.

## 채팅방 만들기

스마트폰에서 텔레그램을 실행시키고, 위에서 만든 텔레그램 봇의 `이름`을 검색하여 채팅방을 만듭니다. 이제부터 이 채팅방에서 `텔레그램 봇`과 통신을 하게 됩니다.

# 실행하기

## 필요 패키지 설치

`Visitor Alarm Telegram`은 `unknown face classifier`에 추가로 다음 패키지를 사용합니다. 아래 명령으로 추가 패키지를 설치하세요.

```bash
(py3) $ pip install python-telegram-bot --upgrade
(py3) $ pip install humanize --upgrade
```

## 실행

`visitor_alarm_telegram.py`에는 아래와 같은 옵션을 줄 수 있습니다. `token` 파라미터는 반드시 들어가야 합니다.

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

위에서 만든 텔레그램 봇의 `token`을 파라미터로 주어 실행을 시키면 아래와 같이 `Visitor Alarm Telegram`이 실행됩니다. `--srcfile` 파라미터를 주지 않았으므로, 디폴트로 웹캠으로부터 얼굴을 인식합니다.

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

스마트폰의 텔레그램 채팅방에서 `/help`라고 입력했을 때 아래와 같이 응답이 나오면 여기까지 성공적으로 온 것입니다.

![help](/assets/img/posts/vat_help.png) 

아래와 같이 PC의 터미널에서 `^C`를 누르면 실행이 종료됩니다.

```bash
^C2020-09-18 18:37:26,066 - Received signal 2 (SIGINT), stopping...
Visitor Alarm Telegram is finished.
```

| Available Commands | Comments |
|--------------------|----------|
| /help | show available commands |
| /settings | show settings |
| /start | start face classifier |
| /stop | stop face classifier |
| /status | show the status of person DB and face classifier |
| /shot | show the current screen of web cam (or the video) |
| /rename old_name new_name | change the person's name |
| /list | list all persons with picture |

# Example Result

Visitor Alarm Telegram을 테스트 하거나 디버깅을 하려면 웹캠 보다는 비디오 파일을 이용하는 것이 편합니다. 여기에서도 비디오 파일을 이용한 테스트 결과를 보여드리겠습니다. 테스트에 사용한 영상은 유튜브에 올려진 영화 '극한직업'의 예고편 파일 입니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xM1CIQd_X4c" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

PC에서 아래와 같이 `--srcfile` 파라미터로 다운받은 비디오 파일을 주고 실행합니다. `--token` 파라미터에 여러분이 만든 Telegram bot의 token을 주어야 합니다.

```bash
$ python visitor_alarm_telegram.py --srcfile ~/Videos/extj.mp4 --token '1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI'
```

## 시작하기

스마트폰의 텔레그램에서 `/start` 를 입력하면 아래와 같이 `Face classifier`가 시작되고 얼굴 인식을 시작합니다.

![start](/assets/img/posts/vat_start.png) 

동시에, PC에서는 아래와 같은 로그가 나옵니다.

```
2020-09-17 16:42:35,022 - Face classifier is started.
* srcfile = /home/rostude/Videos/extj.mp4
* size = 1280x720
* resize_ratio = 1.0 -> 1280x720
* frame_rate = 24.000 f/s
* process every 12 frames
```

## 처음 보는 사람

`Face classifier`가 인식된 얼굴을 처음 보는 사람으로 분류하면, 텔레그램으로 "person_XX appeared first"라는 문구와 함께 그 사람의 얼굴이 전달됩니다. 

![person_first](/assets/img/posts/vat_person_first.png) 

PC에서는 아래와 같은 로그가 출력됩니다.

```bash
2020-09-17 16:42:38,288 - person_01 appeared first
2020-09-17 16:42:42,224 - person_02 appeared first
```

## 재 출현한 사람

이전에 출현한 적이 있는 사람이 다시 출현한 경우에는, 텔레그램으로 "person_XX appeared again"라는 문구와 마지막으로 출현했던 시간 정보와 함께 그 사람의 얼굴이 전달됩니다. 이 때, 왼쪽의 얼굴은 이전에 인식되었던 얼굴이고 오른쪽은 현재 인식된 얼굴 입니다.

![person_again](/assets/img/posts/vat_person_again.png) 

PC에서는 아래와 같은 로그가 출력됩니다.

```
2020-09-17 16:43:04,643 - person_02 appeared again in 22 seconds since 2020-09-17 16:42:40
2020-09-17 16:43:10,124 - person_01 appeared again in 29 seconds since 2020-09-17 16:42:39
```

## 텔레그램 명령어 

![status](/assets/img/posts/vat_status.png) 

텔레그램에서 `/status`를 입력하면 현재의 상황을 보여줍니다. 위 그림을 보니, 지금까지 1분 5초 동안 실행하여 131장의 화면을 분석했고 10 명의 사람이 인식되었다는 것을 알 수 있네요.

![rename](/assets/img/posts/vat_rename.png) 

인식된 사람의 이름은 `person_XX` 같은 이름을 가집니다. `/rename` 명령으로 다른 이름으로 바꿀 수 있습니다. 위 그림에서는 실제 주인공 이름으로 바꿔 주었습니다.

![list](/assets/img/posts/vat_list.png) 

`/list` 명령은 지금까지 인식된 모든 사람의 사진과 이름, 얼굴 수를 보여줍니다. `/rename`으로 바꿔 준 이름이 적용되어 있는 것이 보이시죠?

![shot](/assets/img/posts/vat_shot.png) 

`/shot` 명령은 아주 흥미로운데요. 현재 처리 중인 비디오 화면을 보여줍니다. 위 그림은 테스트 영상의 경우이지만, 웹캠이라면 현재 실시간의 화면을 볼 수 있습니다. 웹캠을 설치한 장소를 때때로 감시하는 용도로 사용할 수 있습니다.

![settings](/assets/img/posts/vat_settings.png) 

`/settings` 명령은 설정 값을 보여줍니다. 

`/stop` 명령은 현재 실행중인 face classifier를 중단합니다.

# Reference

* [Python Face Recognition](/python-face-recognition/)
* [Python Face Clustering](/face-clustering/)
* [Source Code - https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram)
* [Telegram Bot](https://core.telegram.org/bots)
* [Telegram Bot API](https://core.telegram.org/bots/api)
* [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 
