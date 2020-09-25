---
title:  Visitor Alarm Telegram Bot - 텔레그램 봇으로 실시간 방문자 알림
tags:   face-recognition
header:
  teaser: /assets/img/posts/vat_title.png
  image: /assets/img/posts/vat_title.png
date:   2020-09-18 10:00:00 +0900
sidebar:
  - nav: docs
---

텔레그램 봇은 유저와 메시지나 명령/응답을 주고 받을 수 있는 대화형 앱으로, 누구나 무료로 만들 수 있습니다. 이 포스트에서는 비디오에서 인식된 얼굴을 스마트폰으로 전송하는 텔레그램 봇을 만들었습니다. 이 봇을 사용하면, 웹캠을 현관이나 복도 같은 주요 장소에 설치하고, 사람이 인식되면 사용자의 핸드폰으로 실시간 사진을 전송받을 수 있습니다. 얼굴을 인식하고 사람별로 분류하는 기능은 [unknown face classifier](/unknown-face-classifier/)를 이용했고, 여기에 텔레그램 봇 기능을 추가했습니다.

# Visitor Alarm Telegram Bot의 기능

Visitor Alarm Telegram Bot은 다음과 같은 기능을 가집니다.

* 비디오에서 얼굴을 인식
* 인식된 얼굴을 사람 별로 분류. 모르는 사람의 얼굴이라도 비슷한 얼굴끼리 분류
* 처음 보는 사람이 나타나거나, 알고 있는 사람이 다시 나타나면 텔레그램으로 알림을 보냄
* 현재 상태, 실시간 화면 등을 모니터링
* 지금까지 인식된 사람의 리스트를 조회하거나 이름을 변경

이 기능을 활용하면 스마트 초인종, 스마트 CCTV 등을 만들 수 있습니다.

# 텔레그램 봇

텔레그램은 사용자가 직접 [텔레그램 봇](https://core.telegram.org/bots)을 만들 수 있도록 HTTP 기반의 [Telegram Bot API](https://core.telegram.org/bots/api)를 제공합니다. 이를 이용하면 텔레그램 안에서 대화하듯이 Bot에게 명령을 내리고 응답을 받아볼 수 있습니다.

Telegram Bot API는 여러 언어로 포팅이 되어 있습니다. 여기에서는 파이썬으로 포팅된 [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 패키지를 이용하여 만들었습니다. 소스코드는 [GitHub](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram_bot)에 올려져 있습니다.

## 텔레그램 봇 만들기

먼저 사용자의 스마트폰에 텔레그램을 설치합니다. 그리고, 텔레그램 앱에서 BotFather를 통해서 `텔레그램 봇`을 만들어야 합니다. 만드는 방법은 어렵지 않습니다. 구글에서 "[텔레그램 봇 만들기](https://www.google.com/search?newwindow=1&hl=ko&q=%ED%85%94%EB%A0%88%EA%B7%B8%EB%9E%A8+%EB%B4%87+%EB%A7%8C%EB%93%A4%EA%B8%B0)"를 검색하면 많이 나오니, 검색 결과를 참고해서 만드시기 바랍니다.

우리가 사용할 것은 만들어진 텔레그램 봇의 `이름`과 `token` 입니다. 텔레그램 봇의 `이름`은 텔레그램 대화방을 만들 때 사용됩니다. `token`은 텔레그램 봇의 고유번호와 비슷한 것으로, `visitor_alarm_telegram_bot.py`를 실행할 때 입력 값으로 사용됩니다.

## 채팅방 만들기

스마트폰에서 텔레그램을 실행시키고, 위에서 만든 텔레그램 봇의 `이름`을 검색하여 채팅방을 만듭니다. 이제부터 이 채팅방에서 텔레그램 봇과 통신을 하게 됩니다. 아직 텔레그램 봇을 실행시키지 않았기 때문에, 지금은 이 채팅방에 무언가를 입력해도 아무 응답이 없을 것입니다. 

## 메시지 전달 구조

텔레그램 봇은 웹캠이 설치된 PC에서 실행됩니다. 채팅방 안에는 텔레그램 봇과 사용자가 들어가 있습니다. 사용자가 채팅방에 메시지를 올리면, 텔레그램 봇이 그 메시지를 받아 명령을 수행합니다. 텔레그램 봇이 수행한 결과를 채팅방에 메시지로 올리면, 사용자는 그 메시지를 스마트폰에서 받아볼 수 있습니다.

# 텔레그램 봇 실행하기

이제 PC에서 텔레그램 봇을 실행해 봅시다.

## 필요 패키지 설치

python 3.x로 실행해야 하고, virtualenv로 환경을 분리시키기를 추천합니다. 사전에 다음과 같은 패키지가 설치되어 있어야 합니다.

```bash
$ pip install opencv-python
$ pip install opencv-contrib-python
$ pip install dlib
$ pip install face_recognition
$ pip install imutils
$ pip install python-telegram-bot
$ pip install humanize
```

## 실행하고 종료하기

아래와 같이 `visitor_alarm_telegram_bot.py`을 실행해 보세요. `--token` 파라미터는 반드시 들어가야 하고, 위 단계에서 여러분이 만든 token을 주어야 합니다. 문제 없이 실행되었다면 아래와 같은 화면이 나옵니다. 

```bash
$ python visitor_alarm_telegram_bot.py --token '1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI'
Visitor Alarm Telegram Bot is started.
* srcfile = 0 (webcam)
* resize_ratio = 1.0
* sbf (second between frame processed) = 0.5
* similarity threshold = 0.42
* appearance_interval = 10
press ^C to stop...
```

텔레그램 봇을 종료시키고 싶으면, 아래와 같이 PC의 터미널에서 `^C`를 누르면 됩니다. 텔레그램 서버와의 통신 절차 때문에, 종료에는 수 초 정도 시간이 걸립니다.

```bash
^C2020-09-22 10:27:43,615 - Received signal 2 (SIGINT), stopping...
Visitor Alarm Telegram Bot is finished.
```

## 실행 옵션

`visitor_alarm_telegram_bot.py`에는 아래와 같은 옵션을 줄 수 있습니다. 

```bash
(py3) $ python visitor_alarm_telegram_bot.py -h
usage: visitor_alarm_telegram_bot.py [-h] --token TOKEN [--srcfile SRCFILE]
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
* `--sbf` 처리할 비디오 프레임의 시간 간격 (초). PC의 성능이 딸리면 값을 늘려주세요.
* `--resize-ratio` 비디오 크기를 줄여서 처리하도록 할 수 있습니다. 처리 시간이 줄어드는 대신에 정확도는 떨어집니다. PC의 성능이 딸리면 값을 0.5 정도로 줄여주세요.
* `--appearance-interval` 어떤 사람의 얼굴이, 이 시간보다 길게 보이지 않다가 다시 인식되면, 재출현으로 판단합니다.

## 통신 검증

사용자와 텔레그램 봇과의 통신을 확인하기 위해서, 텔레그램 봇을 실행한 다음, 스마트폰의 채팅방에서 `/help`라고 입력해 보세요. 아래와 같이 헬프 화면이 나와야 합니다. 

{:refdef: style="text-align: center;"}
![help](/assets/img/posts/vat_help.png) 
{: refdef}

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

Visitor Alarm Telegram Bot은 웹캠으로 모니터링하는 것이 목적입니다. 하지만, 기능을 테스트 하거나 디버깅을 하려면 비디오 파일을 이용하는 것이 편합니다. 여기에서도 비디오 파일을 이용한 테스트 결과를 보여드리겠습니다. 테스트에 사용한 영상은 유튜브에 올려진 영화 '극한직업'의 예고편 파일 입니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xM1CIQd_X4c" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

이 비디오를 다운 받아서 `~/Videos/extj.mp4`에 저장했고, 아래와 같이 `--srcfile` 파라미터를 주고 실행했습니다.

```bash
$ python visitor_alarm_telegram_bot.py --srcfile ~/Videos/extj.mp4 --token '1234567890:ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHI'
```

## 얼굴 인식 시작하기

텔레그램에서 `/start` 를 입력하면, PC에서 아래와 같은 로그와 함께 face classifier가 시작되고 얼굴 인식을 시작합니다.

```
2020-09-22 10:22:37,428 - Face classifier is started.
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

```
2020-09-22 10:22:40,659 - person_01 appeared for the first time
2020-09-22 10:22:44,551 - person_02 appeared for the first time
```

그리고, 텔레그램으로는 "person_XX appeared for the first time"이라는 문구와 함께 그 사람의 얼굴이 전달됩니다. 

{:refdef: style="text-align: center;"}
![person_first](/assets/img/posts/vat_person_first.png) 
{: refdef}

## 이전에 출현했던 사람이 다시 나타나면

이전에 출현한 적이 있는 사람이 다시 출현한 경우에는, PC에서는 아래와 같은 로그가 출력됩니다.

```
2020-09-22 10:23:06,830 - person_02 appeared again in 22 seconds since 2020-09-22 10:22:43
2020-09-22 10:23:12,310 - person_01 appeared again in 29 seconds since 2020-09-22 10:22:41
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

# 소스코드 설명

소스코드는 [GitHub](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram_bot)에서 받을 수 있으며, 총 3개의 파일로 되어 있습니다. `visitor_alarm_telegram_bot.py`에는 텔레그램 봇이 구현되어 있습니다. `person_db.py`와 `face_classifier.py`는 [unknown face classifier](/unknown-face-classifier/)에 텔레그램 봇을 위한 인터페이스가 조금 추가되었습니다.

| file | 기능 |
|------|------|
| visitor_alarm_telegram_bot.py | 텔레그램 봇 기능 구현 |
| person_db.py | 얼굴과 사람 class를 정의하고, 파일로 저장 |
| face_classifier.py | 얼굴을 인식하고 사람 별로 분류 |

다음에서는 텔레그램 봇 구현의 큰 줄기를 설명합니다.

## Python-telegram-bot

`visitor_alarm_telegram_bot.py`의 텔레그램 봇 기능은 [Python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 패키지를 이용하여 구현되어 있습니다.

[Python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)을 사용하기 위해서 아래와 같이 3개의 import를 합니다.

```python
from telegram.ext import Updater
from telegram.ext import CommandHandler
from telegram.ext import MessageHandler, Filters
```

아래와 같이 Updater와 Bot을 만듭니다. `updater`는 사용자의 명령이나 메시지를 처리하고 응답을 보내는데 사용됩니다. `core`는 채팅방에 알림을 보내는데 사용됩니다. 

```python
# create Telegram Bot
updater = Updater(token=settings.token, use_context=True)
core = telegram.Bot(settings.token)
```

`start_polling()` 함수를 호출하면 텔레그램 봇이 시작됩니다. `idle()` 함수는 유저가 `^C`를 누르기 전까지는 리턴하지 않습니다. 유저가 `^C`를 누르면 텔레그램 서버와 종료 절차를 거친 후 리턴합니다. 

```python
vatb = VisitorAlarmTelegramBot(fc, pdb, args)
vatb.start_polling()
vatb.idle()
```

## 명령어 처리기 (Command pattern)

텔레그램 봇은 (1) 유저가 내린 명령에 대한 응답을 보내거나, (2) 알림을 보낼 수 있습니다. 명령어는 `/`로 시작하는 메시지 입니다. 이런 명령어의 처리는 Command pattern을 이용해서 구현했습니다. 

`CmdDefault` 클래스는 base class이고, 실제 명령어를 처리하는 클래스는 모두 이 클래스를 상속받은 서브클래스 입니다.

```python
class CmdDefault():
    def __init__(self, telegram_bot):
        self.name = self.__class__.__name__[3:].lower()
        self.vatb = telegram_bot

    def usage(self):
        return '/' + self.name

    def method(self, update, context):
        # dummy method - echo
        chat_id = update.effective_chat.id
        reply = update.message.text
        context.bot.send_message(chat_id=chat_id, text=reply)
```

CmdXxx의 이름을 가지는 클래스는 모두 명령어 처리기 입니다. 클래스 이름의 Xxx 부분은 명령어의 `/` 다음 부분과 같습니다. 예를 들어, `CmdRename` 클래스는 `/rename` 명령을 처리합니다.

명령어의 등록은 아래와 같이 `add_command()` 함수로 합니다. 명령어를 등록하면 자동으로 `/help`의 목록이 만들어 집니다.

```python
# add command handlers
self.commands = []
self.add_command(CmdHelp(self))
self.add_command(CmdSettings(self))
self.add_command(CmdStart(self))
self.add_command(CmdStop(self))
self.add_command(CmdStatus(self))
self.add_command(CmdShot(self))
self.add_command(CmdRename(self))
self.add_command(CmdList(self))
```

CmdStart, CmdShot 등 각 명령어의 실제 구현은 소스코드를 참고하세요.

## 알림 보내기 (Observer Pattern)

텔레그램 봇은 특정 이벤트가 발생하는 경우, 유저에게 알림을 보낼 수 있습니다. `FaceClassifier`에서 발생한 이벤트를 `VisitorAlarmTelegramBot`이 수신하는 기능은 observer pattern으로 구현되었습니다. 

`FaceClassifier`는 `Observable` 클래스를 상속받고, 다음과 같은 이벤트를 발생시킵니다.

* Face classifier가 시작되었음
* 얼굴이 처음 보는 사람으로 분류되었음
* 이전에 출현했던 사람이 다시 나타났음
* Face classifier가 종료되었음

위 이벤트를 받아보기 위한 클래스가 `Observer`로, 아래와 같이 정의되어 있습니다.

```python
class Observer():
    # called when Face Classifier is started
    def on_start(self, fc):
        pass

    # called when a person appears first
    def on_new_person(self, person):
        pass

    # called when the person appears again
    def on_person(self, person):
        pass

    # called when Face Classifier is stopped
    def on_stop(self, fc):
        pass
```

`VisitorAlarmTelegramBot`은 `Observer`를 상속받습니다. 다음과 같이 `fc.register_observer(vatb)`를 호출하면 `FaceClassifier`에 Observer로 등록이 됩니다. 

```python
# register 
fc = face_classifier.FaceClassifier(pdb, args)
vatb = VisitorAlarmTelegramBot(fc, pdb, args)
fc.register_observer(vatb)  # register vatb as an observer of fc
```

`FaceClassifier`에 이벤트가 발생하면 `VisitorAlarmTelegramBot`의 해당 함수가 호출되게 됩니다. 실제 구현은, 유저에게 적당한 알림 메시지와 사진을 보내도록 구현되어 있습니다.

# Reference

* [Python Face Recognition](/python-face-recognition/)
* [Unknown Face Classifier](/unknown-face-classifier/)
* [Source Code - https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram_bot](https://github.com/ukayzm/opencv/tree/master/visitor_alarm_telegram_bot)
* [Telegram Bot](https://core.telegram.org/bots)
* [Telegram Bot API](https://core.telegram.org/bots/api)
* [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 
