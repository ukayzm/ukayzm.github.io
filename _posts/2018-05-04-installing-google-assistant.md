---
title:  Installing Google Assistant
date:   2018-05-04 18:40:00 +0900
tags:   voice-assistant raspberry-pi
header:
  image: /assets/img/pexels/gadget-google-google-wifi-1054554.jpg
sidebar:
  - nav: docs
---

구글은 [라즈베리파이에 Google Assistant를 설치할 수 있는 가이드](https://developers.google.com/assistant/sdk/guides/library/python/embed/setup)를 제공하고 있습니다. 이 글에서는 그 가이드를 좀 더 쉽게 설명하겠습니다.

# PC에서 사전 작업

먼저, PC에서 다음과 같이 사전작업을 해야 합니다. Linux나 Windows, MAC 어디에서 수행해도 됩니다.

## Google Could Platform에 계정 가입

GCP 클라우드를 사용하기 위해서 구글 계정에 가입합니다. 기존에 gmail 계정이 있으면 gmail 계정을 사용하면 됩니다. [http://www.google.com/cloud](http://www.google.com/cloud)로 가서, 좌측 상단에 Try it Free 버튼을 눌러서 구글 클라우드에 가입합니다.

다음 콘솔에서 상단의 Google Cloud Platform 을 누르면 좌측에 메뉴가 나타나는데, 메뉴 중에서 “결제" 메뉴를 선택한후 결제 계정 추가를 통해서 개인 신용 카드 정보를 등록합니다. Google Assistant는 무료로 설치/사용할 수 있으므로, 신용 카드 등록을 해도 돈이 결제되지는 않습니다.

## 구글 프로젝트 생성

[https://console.cloud.google.com](https://console.cloud.google.com)에서 프로젝트를 생성합니다. 프로젝트는 구글 클라우드에서 지원하는 서비스를 하나로 묶어서 관리하는 집합이라고 볼 수 있습니다. 아래 화면은 구글 클라우드 콘솔의 맨 위에 위치한 바입니다. 필자가 만든 VaGoogle1 이라는 이름의 프로젝트가 현재 활성화되어 있는 것을 볼 수 있습니다.

![GCP-project](/assets/img/posts/GCP-project.png)

## Google Assistant API 활성화

다음 단계는 새로 생성한 프로젝트에 Google Assistant API를 활성화하는 것입니다.

페이지 좌상단 ☰ > APIs & Services > Dashboard를 눌러 API 관리 화면으로 들어갑니다. 위쪽의 “ENABLE APIS AND SERVICES”를 누른 후 API 검색창에서 “Google Assistant API”를 검색합니다. 또는 프로젝트 콘솔의 맨 위 검색창에서 검색해도 됩니다. 검색된 “Google Assistant API”를 선택하여  API 관리 화면으로 들어갑니다. “ENABLE”을 누르면 이 프로젝트에서 Google Assistant API를 사용할 수 있게 됩니다.  Google Cloud Platform 페이지에서 왼쪽 상단의 ☰ > APIs & Services > Dashboard를 눌러서 현재 프로젝트에서 사용 가능한 API 리스트 안에 Google Assistant API가 있는지를 확인합니다.

## OAuth Client ID 생성

활성화한 Google Assistant API를 사용하려면 Consent screen과 OAuth Client ID를 만들어야 합니다. OAuth client ID는 라즈베리파이에서 구동되는 Google Assistant가 서버에 OAuth 2.0 access token을 요청할 때 사용됩니다.
☰ > APIs & Services > Credential > OAuth consent screen을 누릅니다. 적당히 Product name을 입력합니다. 필자는 DIYGoogleAssistant로 정했습니다. SAVE를 누른 후, Client ID를 설정합니다. 적당한 client ID를 입력합니다. 필자는 diy_ga_client를 입력하였습니다. Type은 반드시 Other로 합니다. Create를 누르고 OK를 누르면 OAuth Client ID가 만들어집니다. 만들어진 client ID는 아래 그림과 같이, ☰ > APIs & Services > Credential > Credentials에서 확인할 수 있습니다.

![GCP-credentials](/assets/img/posts/GCP-credentials.png)

Client ID의 맨 오른쪽에 있는 아래 화살표를 눌러서, JSON file을 다운로드 합니다. 파일 이름은 client_secret_<client_id>.json 입니다. 이 파일을 라즈베리 파이의 /home/pi 디렉토리에 복사해야 합니다. 중요한 점은, 파일 이름이 변경되면 안됩니다. 아래와 같이 scp로 복사할 수 있습니다.
```
$ scp ~/Downloads/client_secret_client-id.json pi@raspberry-pi-ip-address:/home/pi/
```
또는, 단순한 텍스트 파일이므로, putty 접속 창에서 텍스트 붙여넣기를 해도 됩니다.

> [주의] client secret 파일의 이름이 변경되거나 위치가 /home/pi가 아니면 모델 등록시에 에러가 발생합니다. 글을 쓰는 시점에 googlesamples-assistant-devicetool 명령의 --client-secrets 옵션이 정상 동작하지 않았습니다.

## 음성 데이터 사용 허용

그 다음으로, Google에게 우리가 생성한 음성 데이터를 사용하도록 허용하여, Google이 음성 데이터를 분석할 수 있도록 해야 합니다.
[https://myaccount.google.com/activitycontrols](https://myaccount.google.com/activitycontrols)에 접속합니다. 그리고, 아래 항목을 켭니다.

* Web & App Activity
  * 이 화면에서 체크박스에 체크도 해야 합니다. “Include Chrome browsing history and activity from websites and apps that use Google services”
* Device Information
* Voice & Audio Activity

# 라즈베리파이에서 작업

## 필요 라이브러리 설치

먼저, 아래 명령으로 Google Assistant가 사용하는 라이브러리를 설치합니다.

```
$ sudo apt-get update
$ sudo apt-get install portaudio19-dev libffi-dev libssl-dev
```

## Virtual Environment 생성

Python은 버전별로 의존성 문제가 있을 수 있으므로, 별도의 환경을 만들어 실행시키는 것이 일반적입니다. 라즈베리파이에 ssh로 접속한 후, 아래 명령으로 Google Assistant를 실행하기 위한 python 환경을 만듭니다.

```
$ cd ~
$ sudo apt-get install python3-dev python3-venv
$ python3 -m venv py3
$ py3/bin/python -m pip install --upgrade pip setuptools
```

간혹 python3-venv가 없다는 에러가 나는 경우가 있는데,  python3.4-venv로 바꿔서 시도하면 성공합니다.

위 작업은 “py3”라는 이름의 python 가상 환경을 만든 것입니다. 이 이름은 독자가 자유롭게 바꿀 수 있습니다. 아래 명령으로 “py3” 가상환경을 활성화 하거나 비활성화 할 수 있습니다.

```
$ source py3/bin/activate      # 가상환경 활성화
(py3) $
(py3) $ deactivate             # 가상환경 비활성화
$
```

source py3/bin/activate로 가상환경을 활성화 시키면 프롬프트 앞에 (py3)라는 가상환경 이름이 붙게 되어 현재 활성화된 가상환경을 알 수 있습니다. 앞으로 Google Assistant는 이 가상환경 위에서 동작할 것입니다.

## Google Assistant 설치

Python 가상환경 위에서 아래와 같이 Google Assistant library, SDK와 oauthtool을 설치합니다.

```
(py3) $ python -m pip install --upgrade google-assistant-library
(py3) $ python -m pip install --upgrade google-assistant-sdk[samples]
(py3) $ python -m pip install --upgrade google-auth-oauthlib[tool]
```

혹시 위 과정에서 오류가 발생하면 다음 명령을 수행한 후 다시 시도해 보세요.

```
(py3) $ python -m pip install wheel
```

## Credential 생성

```
(py3) $ google-oauthlib-tool --scope https://www.googleapis.com/auth/assistant-sdk-prototype --save --headless --client-secrets /home/pi/client_secret_312310234098-ard5v97s9j04qjm2q6a27pgg5obidp9b.apps.googleusercontent.com.json
```

위 명령을 수행하면 아래와 같은 내용이 출력됩니다.

```
Please visit this URL to authorize this application: https://accounts.google.com/o/oauth2/auth?response_type=code...이하_생략...
Enter the authorization code:
```

PC의 웹브라우저에서 https:// 이하 URL을 입력하여 해당 페이지로 이동합니다. PC 단계에서 사용한 계정을 선택하고, 다음 화면에서 “ALLOW”를 누르면 아래와 같이 code가 출력됩니다.

![GA-authorization-code](/assets/img/posts/GA-authorization-code.png)

이 코드를 복사한 다음, google-oauthlib-tool을 실행시킨 터미널에 붙여넣기를 합니다.

```
Enter the authorization code: 4/Sn_8u9QZc8oamYcLz9bzIzZpk_yjI0fat5hHLY3CxqA
credentials saved: /home/pi/.config/google-oauthlib-tool/credentials.json
(py3) $
```

Credential 생성에 성공하면 위와 같이, “credentials saved” 가 출력됩니다.

## 디바이스 등록

아래와 같이 googlesamples-assistant-devicetool를 실행하여 디바이스를 등록합니다.

```
(py3) $ googlesamples-assistant-devicetool register-model --manufacturer "DIY co." --product-name "GAonPI" --description "my own google assistant" --type LIGHT --model "No1"
Creating new device model
Model No1 successfully registered
(py3) $ googlesamples-assistant-devicetool list --model
Device Model Id: No1
        Project Id: possible-dream-193223
        Device Type: action.devices.types.LIGHT
No traits
```

여기까지 했으면 모든 작업이 완료된 것입니다.

# 테스트

아래 명령으로 데모용 Google Assiatant를 실행시켜 봅니다.

```
(py3) $ google-assistant-demo --device_model_id No1
device_model_id: No1
device_id: 326B5D34B14FB4F949F4DC8A52B2E561

ON_MUTED_CHANGED:
  {'is_muted': False}
ON_START_FINISHED
```

“Hey Google”이라고 말해보세요. 띠딩 소리가 나면서, 아래와 같이 출력됩니다.

```
ON_CONVERSATION_TURN_STARTED
```

“What is the weather today?”라고 말해보세요. 그러면 아래와 같이 출력되면서 현재 위치의 날씨를 말해 줍니다.

```
ON_END_OF_UTTERANCE
ON_RECOGNIZING_SPEECH_FINISHED:
  {'text': 'what is the weather today'}
ON_RESPONDING_STARTED:
  {'is_error_response': False}
ON_RESPONDING_FINISHED
ON_CONVERSATION_TURN_FINISHED:
  {'with_follow_on_turn': False}

^C
(py3) $
```

성공적으로 테스트를 마쳤습니다.

위와 같이 ^C를 누르면 Google Assistant Demo app에서 빠져나옵니다.

나중에 라즈베리파이를 껐다 다시 켠 이후에라도 아래 명령으로 Google Assiatant를 다시 실행할 수 있습니다.

```
$ source py3/bin/activate
(py3) $ google-assistant-demo --device_model_id No1
```

# 참고 사이트

* [Google's Official Set Up Guide](https://developers.google.com/assistant/sdk/guides/library/python/embed/setup)
