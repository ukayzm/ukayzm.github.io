---
title:  AWS IAM 사용자 만들기
tags:   cloud
header:
  teaser: /assets/img/iam_admin_user/AWS-IAM-Exploitation.png
  image: /assets/img/iam_admin_user/AWS-IAM-Exploitation.png
date:   2021-04-18 10:00:00 +0900
sidebar:
  - nav: docs
---

AWS를 시작하면서 제일 처음 해야 할 일은 당연히 AWS 계정을 만드는 일입니다. 그 다음 해야 하는 일은 IAM에서 사용자를 생성하는 것입니다. 엥? AWS 계정이 있는데, IAM 사용자를 또 만들어야 한다구요? 네, 그렇습니다. 이 글을 끝까지 읽어 보시면 그 이유와 용도를 알 수 있습니다.

# AWS IAM

IAM은 AWS Identity and Access Management의 약자로, 사용자 및 그룹을 만들어서 AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 웹 서비스입니다. 

AWS에는 두 가지 유형의 사용자(계정)가 있습니다. 

* 루트 사용자 - AWS 계정이 생성될 때 만들어집니다. 맨 처음 AWS에 가입할 때 만든 계정이 바로 루트 사용자 입니다. AWS의 루트 사용자는 모든 리소스에 대한 모든 액세스가 가능합니다.
* IAM 사용자 - 루트 사용자 또는 IAM 관리자가 만듭니다. 각 사용자 별로 AWS 서비스 또는 리소스 액세스에 제한을 둘 수 있습니다. 액세스 가능한 수준을 그룹으로 만들고 각 사용자를 적당한 그룹에 등록하는 방식으로 제한을 제어합니다.

이론상 IAM 사용자를 만들지 않고 루트 사용자로 모든 서비스를 이용할 수도 있겠습니다만, 이것은 아주 위험한 방식입니다. 대신에 AWS에서는 IAM 사용자를 별도로 만들기를 "강력하게" 권고하고 있습니다. 루트 사용자는 맨 처음 관리자 계정을 생성할 때에만 사용하고, 그 다음부터는 IAM 사용자로 이용하는 것이 모범적인 이용 방법 입니다. 그런 다음 루트 사용자 자격 증명을 안전하게 보관해 두고, 몇 가지 계정 및 서비스 관리 작업을 수행할 때만 해당 자격 증명을 사용하는 것이 좋습니다.

## 루트 사용자 자격 증명이 필요한 작업

아래와 같은 작업은 루트 사용자만 할 수 있습니다.

* 계정 설정의 변경
* IAM 사용자 권한의 복원
* 결제 및 비용 관리 콘솔에 대한 IAM 액세스 활성화
* 특정 세금 계산서 조회
* AWS 계정 닫기
* AWS 지원 플랜 변경 또는 취소
* 마켓플레이스에 판매자로 등록
* Amazon S3 버킷을 구성하여 MFA 삭제를 활성화
* 잘못된 VPC ID 또는 VPC 엔드포인트 ID가 들어있는 Amazon S3 버킷 정책을 편집 또는 삭제
* GovCloud 등록

위에 나열된 작업 외의 모든 작업은 루트 사용자가 아닌 IAM 사용자로 하는 것이 좋습니다. 심지어, 다른 IAM 사용자를 추가하는 작업도 IAM 사용자가 하는 것이 좋습니다. 

이 글에서는 루트 사용자 수준의 권한을 가진 관리용 IAM 사용자를 만들 것입니다. 그리고, 관리용 IAM 사용자를 이용해서 개발용 IAM 사용자를 만들어 보겠습니다.

# 사전 작업

아직은 IAM 사용자를 만들지 않은 상태이므로, 첫번째 IAM 사용자를 만드는 작업은 루트 사용자가 해야 합니다. 

## 루트 사용자로 로그인

[IAM 콘솔](https://console.aws.amazon.com/iam/)에 접속해서 아래와 같이 [루트 사용자]로 로그인 합니다.

<figure>
    <a href="/assets/img/iam_admin_user/01_login.png" class="align-center"><img src="/assets/img/iam_admin_user/01_login.png"></a>
</figure>

## 결제 데이터 액세스 활성화

그 다음, 생성할 IAM 관리자에 대한 결제 데이터 액세스를 활성화해야 합니다.

탐색 표시줄에서 계정 이름을 선택한 다음 내 계정을 선택하여 계정 설정 페이지를 새로 엽니다.

<figure>
    <a href="/assets/img/iam_admin_user/02_my_account.png" class="align-center"><img src="/assets/img/iam_admin_user/02_my_account.png"></a>
</figure>

계정 설정 페이지에서 아래로 스크롤을 하면 결제 정보에 대한 IAM 사용자 및 역할 액세스 항목이 나옵니다. 오른쪽의 "편집"을 누릅니다.

<figure>
    <a href="/assets/img/iam_admin_user/03_edit.png" class="align-center"><img src="/assets/img/iam_admin_user/03_edit.png"></a>
</figure>

"IAM 액세스 활성화"를 누르고 "업데이트"를 누릅니다.

<figure>
    <a href="/assets/img/iam_admin_user/04_activate.png" class="align-center"><img src="/assets/img/iam_admin_user/04_activate.png"></a>
</figure>

그러면 아래 쪽에 문구가 "결제 정보에 대한 IAM 사용자/역할 액세스가 활성화되었습니다."로 바뀝니다.

# 관리용 IAM 사용자 만들기

탐색 표시줄에서 서비스를 선택한 다음, IAM을 선택해 [IAM 대시보드](https://console.aws.amazon.com/iam/)로 돌아갑니다.

<figure>
    <a href="/assets/img/iam_admin_user/05_to_iam.png" class="align-center"><img src="/assets/img/iam_admin_user/05_to_iam.png"></a>
</figure>

## 사용자 추가 시작하기

IAM 대시보드의 탐색창에서 "사용자"를 누르고 "사용자 추가"를 누릅니다.

<figure>
    <a href="/assets/img/iam_admin_user/06_add_user.png" class="align-center"><img src="/assets/img/iam_admin_user/06_add_user.png"></a>
</figure>

[사용자 이름]을 입력합니다. 관리자 역할을 할 사용자를 추가하는 것이므로, 저는 헷갈리지 않도록 Administrator를 입력했습니다. 

액세스 유형은 AWS Management Console 액세스 옆의 확인란을 선택합니다. 이렇게 하면 AWS Management Console (웹페이지)로 로그인을 할 수 있습니다. 

사용자 지정 암호를 선택한 다음 텍스트 상자에 새 암호를 입력합니다. 필요시, 사용자가 다음에 로그인할 때 비밀번호를 변경하도록 설정할 수 있습니다.

"다음:권한"을 누릅니다. 

<figure>
    <a href="/assets/img/iam_admin_user/07_add_user2.png" class="align-center"><img src="/assets/img/iam_admin_user/07_add_user2.png"></a>
</figure>

## 그룹 생성

처음 사용자를 추가하는 것이므로, 당연히 그룹도 새로 만들어야 합니다. 그룹은 말 그대로 사용자의 집합 입니다. "그룹 생성"을 누릅니다.

<figure>
    <a href="/assets/img/iam_admin_user/08_create_group.png" class="align-center"><img src="/assets/img/iam_admin_user/08_create_group.png"></a>
</figure>

그룹 생성 대화 상자의 그룹 이름에 "Administrators"를 입력합니다. 유저를 만들 때는 단수형을 썼지만, 그룹 이름은 끝에 "-s"를 붙여서 복수형을 썼다는 것, 알아 채셨나요?

AdministratorAccess 정책을 추가합니다. 정책이란, 어떤 리소스에 어떤 액세스를 가능하게 할 지 권한을 정의해 놓은 것입니다. 일단은, AdministratorAccess 정책은 모든 리소스에 모든 액세스가 가능하다고 알고 넘어갑시다. 

<figure>
    <a href="/assets/img/iam_admin_user/09_create_group2.png" class="align-center"><img src="/assets/img/iam_admin_user/09_create_group2.png"></a>
</figure>

"그룹 생성"을 누르면 "Administrators" 그룹이 생성됩니다. 그룹 목록이 있는 페이지로 돌아가는데, 목록에 새 그룹이 표시되지 않으면 "새로 고침"을 하면 됩니다. "Administrators" 그룹을 선택하고 "다음: 태그"를 누르면 사용자를 그룹에 포함시키고 다음 화면으로 넘어갑니다.

<figure>
    <a href="/assets/img/iam_admin_user/10_add_group.png" class="align-center"><img src="/assets/img/iam_admin_user/10_add_group.png"></a>
</figure>

다음은 태그 추가 화면이 나옵니다. 태그는 사용자 또는 AWS 리소스에 뭍이는 이름이라고 생각할 수 있습니다. 선택 사항이므로, 일단은 "다음: 검토"를 눌러서 넘어갑니다. 

## 사용자 추가 완료

위에서 만든 사용자 이름과 그룹 이름을 확인합니다. "사용자 만들기"를 누르면 새 사용자가 만들어집니다.

<figure>
    <a href="/assets/img/iam_admin_user/11_create_user.png" class="align-center"><img src="/assets/img/iam_admin_user/11_create_user.png"></a>
</figure>

아래와 같은 화면이 나오고, 사용자 추가 작업이 완료됩니다.

<figure>
    <a href="/assets/img/iam_admin_user/12_user_created.png" class="align-center"><img src="/assets/img/iam_admin_user/12_user_created.png"></a>
</figure>

"마지막 기회" 어쩌구 하는 문구가 나오는데, 위에서 액세스 유형을 "AWS Management Console 액세스"로 했기 때문에 큰 의미는 없습니다. 그저 앞에서 입력한 비밀번호를 잘 기억하는 것으로 충분합니다. 

## 새 사용자로 로그인

로그인 URL을 사용하여 새 사용자로 로그인 할 수 있습니다. 로그인 URL은 `https://xxxxxxxxxxxx.signin.aws.amazon.com/console` 처럼 생겼습니다. `xxxxxxxxxxxx` 부분은 12자리 숫자인 계정 ID 입니다. AWS가 자동으로 부여하고, 사용자마다 다릅니다.

로그인 URL은 세 가지 방법으로 알 수 있습니다.

1. 위 그림에 나온 링크
2. .csv 파일을 다운로드 받으면 안에 링크가 있습니다.
3. "이메일 전송"을 누르면 아래와 같은 내용의 이메일로 링크를 전송할 수도 있습니다.

<figure>
    <a href="/assets/img/iam_admin_user/14_email.png" class="align-center"><img src="/assets/img/iam_admin_user/14_email.png"></a>
</figure>

로그인 URL로 접속하면 아래와 같은 화면이 나옵니다. 계정 ID 12자리는 이미 입력이 되어 있을 것입니다. 사용자 이름(Administrator)과 암호를 입력하여 로그인 합니다.

<figure>
    <a href="/assets/img/iam_admin_user/13_login_as_administrator.png" class="align-center"><img src="/assets/img/iam_admin_user/13_login_as_administrator.png"></a>
</figure>

또는 보통의 AWS 로그인 화면에서 "IAM 사용자"를 선택한 다음, 계정 ID를 수동으로 입력해도 됩니다.

Administrator 사용자로 로그인하고 맨 위에 있는 바를 보면, 아래 그림과 같이 사용자 이름이 Administrator로 되어 있는 것을 볼 수 있습니다.

<figure>
    <a href="/assets/img/iam_admin_user/15_logged_in_as_administrator.png" class="align-center"><img src="/assets/img/iam_admin_user/15_logged_in_as_administrator.png"></a>
</figure>

# 관리용 사용자로 개발용 IAM 사용자 만들기

"Administrator"라는 사용자는 "AdministratorAccess" 정책을 가진 "Administrators" 그룹에 속해 있으므로 AWS의 거의 모든 서비스를 이용할 수 있습니다. 그러므로, 이제부터는 특별한 경우를 제외하고는 루트 사용자 대신에 Administrator 사용자로 관리 업무를 할 수 있습니다. 

여기에서는 PowerUser라는 사용자를 만들어 보겠습니다. 이 사용자는 모든 서비스에 액세스 할 수 있지만 유저 관리는 할 수 없는 권한을 부여할 것입니다. 그리고, 액세스 키 ID와 비밀 액세스 키를 받아서, AWS API, CLI, SDK 및 기타 개발 도구에서 사용할 수 있도록 할 것입니다.

## 프로그래밍 방식 액세스 유형으로 사용자 추가

Administrator로 로그인 하고, [IAM 콘솔](https://console.aws.amazon.com/iam/)에 접속합니다. 

왼쪽에 있는 IAM 대시보드의 탐색창에서 “사용자”를 누르고 “사용자 추가”를 누릅니다.

아래 그림과 같이 사용자 이름을 입력합니다. 그리고, "프로그래밍 방식 액세스"에 체크를 합니다. 그래야 액세스 키 ID와 비밀 액세스 키를 받을 수 있습니다.

<figure>
    <a href="/assets/img/iam_admin_user/16_cli_access.png" class="align-center"><img src="/assets/img/iam_admin_user/16_cli_access.png"></a>
</figure>

## 파워 유저 그룹 생성

위에서는 모든 권한이 부여된 Administrators 그룹을 생성했었습니다. 이번에는 유저 관리는 할 수 없지만 다른 모든 서비스에는 액세스가 가능한 PowerUsers란 그룹을 만들겠습니다. 초기 개발 용도에 딱 맞겠네요. 나중에 조직이 커지면 서비스별로 세분화해서 그룹을 생성할 수도 있습니다.

"그룹 생성"을 누릅니다. 그룹 이름은 PowerUsers로 했습니다. 그룹 이름이므로 복수형을 사용했습니다. 

이미 만들어져 있는 정책은 800개가 넘는데요. 그 중에서 PowerUserAccess라는 정책을 추가할 것입니다. 찾기에서 "poweruseraccess"를 입력한 다음, PowerUserAccess 옆에 체크를 하고 "그룹 생성"을 누릅니다.

<figure>
    <a href="/assets/img/iam_admin_user/17_power_user_access.png" class="align-center"><img src="/assets/img/iam_admin_user/17_power_user_access.png"></a>
</figure>

## 사용자 추가 완료

태그 화면을 건너 뛰고, "사용자 만들기"를 누르면 PowerUser 사용자가 만들어 집니다.

<figure>
    <a href="/assets/img/iam_admin_user/18_poweruser_created.png" class="align-center"><img src="/assets/img/iam_admin_user/18_poweruser_created.png"></a>
</figure>

**[중요]** 방금 만든 사용자는 "프로그래밍 방식 액세스" 유형으로 만들었습니다. 프로그래밍 방식으로 액세스 하려면 "액세스 키 ID"와 "비밀 액세스 키"가 필요합니다. 그런데, **"비밀 액세스 키"는 이 화면에서만 받을 수 있습니다.** 그러므로, 여기에서 .csv 파일을 다운로드 받아서 잘 보관해 두거나, "표시"를 눌러서 비밀 액세스 키를 복사해 두어야 합니다. 

만약 이 단계에서 "비밀 액세스 키"를 못받은 경우에는 액세스 키를 다시 만들어야 합니다. IAM > 사용자 > 보안 자격 증명 으로 가서 액세스 키를 다시 만들 수 있습니다. 

"액세스 키 ID"와 "비밀 액세스 키"는 추후 AWS 서비스를 개발하는데 두고 두고 사용되는 아주 중요한 정보입니다.

## 액세스 키 설치하기

액세스 키는 `~/.aws/credintials` 파일에 설치합니다. 설치된 액세스 키를 확인하려면 `awscli` 패키지가 필요합니다.

먼저 아래 명령으로 `awscli` 패키지를 설치합니다.

```shell
$ sudo apt install awscli
```

위에서 다운로드 받은 .csv 파일에 있는 액세스 키를 아직 설치하지 않았습니다. 그러므로, 아래와 같이 `aws configure list` 명령을 내리면 'not set' 이라고 나옵니다.

```shell
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key                <not set>             None    None
secret_key                <not set>             None    None
    region                <not set>             None    None
```

aws 관련 명령을 내려도 아래와 같이 credentials를 찾을 수 없다는 에러가 나옵니다.

```shell
$ aws s3 cp s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website ./ --recursive
fatal error: Unable to locate credentials
```

이제, 액세스 키를 설치하기 위해서, `~/.aws` 디렉토리를 만들고 `credentials`과 `config` 두 개의 파일을 만들어 줍니다.

`aws_access_key_id`와 `aws_secret_access_key`는 바로 위에서 다운로드 받은 `PowerUser_credentials.csv`에 있는 값을 넣어줍니다.


```shell
$ vi ~/.aws/credentials
[default]
aws_access_key_id=ABCD2MQOOQVDDQHAAMUF
aws_secret_access_key=ABCDoit5VBkZeA1VxE8ipj4rWeKYSS1O1lrWHSGH
```

`config` 파일의 region에는 서울 리전을 뜻하는 `ap-northeast-2`을 넣습니다.

```shell
$ vi ~/.aws/config
[default]
region=ap-northeast-2
output=json
```

그러고 나서 `aws configure list` 명령을 내리면 아래와 같이 설치된 액세스키와 비밀키가 표시됩니다.

```shell
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************AMUF shared-credentials-file
secret_key     ****************HSGH shared-credentials-file
    region           ap-northeast-2      config-file    ~/.aws/config
```

# 만들어진 사용자 확인하기

IAM > 사용자에서 아래와 같이 Administrator와 PowerUser 두 개의 사용자가 추가되었음을 확인할 수 있습니다. Administrator는 비밀번호가 있고, PowerUser는 액세스 키가 있는 것이 보이시죠?

<figure>
    <a href="/assets/img/iam_admin_user/19_users.png" class="align-center"><img src="/assets/img/iam_admin_user/19_users.png"></a>
</figure>

IAM > 그룹에서 아래와 같이 Administrators와 PowerUsers 두 개의 그룹이 추가되었음을 확인할 수 있습니다.

<figure>
    <a href="/assets/img/iam_admin_user/20_groups.png" class="align-center"><img src="/assets/img/iam_admin_user/20_groups.png"></a>
</figure>

# 요약

* IAM에서는 사용자 및 그룹을 관리할 수 있습니다.
* 루트 사용자는 AWS 계정을 처음 생성할 때 만들어진 계정입니다. 아주 중요한 계정이므로 특별한 경우에만 사용하고, 대부분은 IAM 사용자를 만들어서 사용하는 것을 강력 권장합니다.
* 여기에서는 두 개의 IAM 사용자를 만들었습니다.
  - Administrator - 관리자 권한
  - PowerUser - 개발자 용도
* PowerUser는 프로그래밍 방식 액세스 유형으로 만들었기 때문에, "액세스 키 ID"와 "비밀 액세스 키"를 부여받았습니다.
