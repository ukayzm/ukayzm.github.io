---
title:  AWS Cognito - 로그인 관리
tags:   cloud
header:
  teaser: /assets/img/aws_cognito/cognito.jpg
  image: /assets/img/aws_cognito/cognito.jpg
date:   2021-05-06 20:00:00 +0900
sidebar:
  - nav: docs
---

요즘의 웹페이지나 모바일앱은 가입을 하고 로그인 기능을 제공하는 곳이 대부분입니다. AWS에도 당연히 이런 기능을 손쉽게 구현할 수 있는 서비스를 제공합니다. AWS Cognito가 그것입니다. 

AWS Cognito에는 사용자 풀과 자격 증명 풀 두 가지가 있습니다.

* 사용자 풀 - 앱 사용자의 가입 및 로그인 옵션을 제공하는 사용자 모음
* 자격 증명 풀 -  다른 AWS 서비스에 대한 사용자 액세스 권한을 부여

구체적으로는, 아래와 같은 시나리오로 진행됩니다.

1. 첫 번째 단계에서 앱 사용자는 사용자 풀을 통해 로그인하여 인증 성공 이후 사용자 풀 토큰을 받습니다.
2. 다음으로 앱은 자격 증명 풀을 통해 사용자 풀 토큰을 AWS 자격 증명으로 교환합니다.
3. 마지막으로 앱 사용자는 AWS 자격 증명을 사용하여 Amazon S3 또는 DynamoDB 등 기타 AWS 서비스에 액세스할 수 있습니다.

{:refdef: style="text-align: center;"}
![scenario-cup-cib2](/assets/img/aws_cognito/scenario-cup-cib2.png) 
{: refdef}

복잡해 보이지만, 복잡한 기능은 AWS에서 알아서 자동으로 해 줍니다. 실제 구현을 위해서 AWS에서 SDK도 제공합니다. 

프리티어의 경우, 월별 50000 유저까지는 무료입니다.

이 글에서는 AWS Cognito에서 사용자 풀을 만들고, 웹페이지에 로그인 기능을 만들어 보겠습니다. 작업 순서는 아래와 같습니다.

1. [AWS Cognito Console](https://console.aws.amazon.com/cognito/home)에서 사용자 풀 만들기
2. 앱 클라이언트 추가하기
3. 예제 웹페이지의 javascript 코드에 사용자 풀 ID와 앱 클라이언트 ID 설정
4. 예제 웹사이트에서 가입(Sign up)하고 로그인(Sign in)해 보기
5. AWS Console에서 등록된 사용자 확인

작업은 리눅스 머신에 접속한 터미널과 AWS 웹 콘솔, 두 군데에서 할 것입니다. 

# 사전 작업

* 관리자용 IAM 사용자를 만드셨나요? 아니라면, 먼저 [여기](/aws-create-iam-user/)에서 관리자용 IAM 사용자를 만들기를 강력히 추천합니다.
* 만들어진 사용자 풀을 실제로 검증해 보기 위해서, AWS에서 제공하는 예제 웹페이지를 사용할 것입니다. [여기](/aws-amplify/)에서 웹페이지를 만드시기 바랍니다.

# 사용자 풀 만들기

사용자 풀이란 말 그대로 사용자의 모음입니다. 사용자 별로 ID, 암호, e-mail, 전화번호 등의 정보를 저장하고, Facebook, Google, Amazon, Apple ID와 연동할 수 있습니다. 암호 규칙을 정할 수 있고, 사용자가 필수로 입력해야 하는 필드를 지정하고, 선택적으로 주소, 전화번호 등을 저장할 수도 있습니다.

사용자 풀을 만들기 위해서는 관리자용 IAM 유저로 로그인해야 합니다. 좌상단에서 서비스를 클릭한 다음 "보안, 자격 증명 및 규정 준수" 아래에서 Cognito를 선택하거나, [AWS Cognito Console](https://console.aws.amazon.com/cognito/home)로 바로 들어갑니다.

아래와 같은 화면이 나왔나요? "사용자 풀 관리"를 누릅니다.

{:refdef: style="text-align: center;"}
![01_pool](/assets/img/aws_cognito/01_pool.png) 
{: refdef}

아직 사용자 풀을 만들지 않았으므로, "사용자 풀 생성" 외에 다른 것은 나오지 않을 것입니다. "사용자 풀 생성"을 눌러 만들기를 시작합니다.

{:refdef: style="text-align: center;"}
![02_create_pool](/assets/img/aws_cognito/02_create_pool.png) 
{: refdef}

다음 화면에서 사용자 풀의 이름을 입력합니다. 저는 AWS의 예제 이름인 WildRydes 라고 입력했습니다. 입력 후, "기본값 검토"를 누릅니다.

{:refdef: style="text-align: center;"}
![03_name](/assets/img/aws_cognito/03_name.png) 
{: refdef}

기본값 검토 화면이 나옵니다. 여기서 중요한 것은 "속성" 입니다. 아래 화면의 연필 아이콘을 눌러 속성 중에서 바꿀 것이 있는지를 확인합니다.

{:refdef: style="text-align: center;"}
![04_1_attribute](/assets/img/aws_cognito/04_1_attribute.png) 
{: refdef}

속성에서 다음과 같은 것을 지정할 수 있습니다.
* 로그인 ID - 이름/이메일/전화번호/별칭, 대소문자 구분 여부
* 필수 입력 사항 - 주소, 생일, 성별, 전화번호 등등
* 그 외, 사용자 지정 속성 (string or number type)

{:refdef: style="text-align: center;"}
![04_2_attr2](/assets/img/aws_cognito/04_2_attr2.png) 
{: refdef}

{:refdef: style="text-align: center;"}
![04_3_attr3](/assets/img/aws_cognito/04_3_attr3.png) 
{: refdef}

일단 사용자 풀을 만들고 나면 "속성"은 더 이상 수정할 수 없습니다. 그러므로 신중하게 선택해야 합니다. 이후에 속성을 변경해야 하는 일이 생긴다면, 새 사용자 풀을 만든 다음 기존 사용자 풀을 새 사용자 풀로 마이그레이션 해야 합니다. 구체적인 방법은 [여기](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cognito-change-user-pool-attributes/)를 참고하세요.

이 글에서는 최대한 단순하게 하기 위해서, email만 받도록 했습니다.

속성을 정했으면 "풀 생성"을 눌러서 사용자 풀을 생성합니다.

{:refdef: style="text-align: center;"}
![04_create_pool](/assets/img/aws_cognito/04_create_pool.png) 
{: refdef}

사용자 풀이 생성되면 아래와 같이 사용자 풀 ID가 나옵니다. 여기서 중요한 정보는 "사용자 풀 ID" 입니다.

{:refdef: style="text-align: center;"}
![05_pool_id](/assets/img/aws_cognito/05_pool_id.png) 
{: refdef}

이제 다시, [AWS Cognito Console](https://console.aws.amazon.com/cognito/home)로 들어가서 "사용자 풀 관리"를 누르면, 아래와 같이 방금 만든 사용자 풀이 출력됩니다.

{:refdef: style="text-align: center;"}
![06_pool_created](/assets/img/aws_cognito/06_pool_created.png) 
{: refdef}

# 앱 클라이언트 추가하기

사용자 풀을 생성했으면, 이제 사용자 풀에 접근할 수 있는 클라이언트를 추가해야 합니다. 추가한 클라이언트에는 앱 클라이언트 ID가 부여됩니다. 여기에서 만든 앱 클라이언트 ID를 가진 앱만 사용자 풀에 접근할 수 있습니다.

[AWS Cognito Console](https://console.aws.amazon.com/cognito/home)로 들어가서, "사용자 풀 관리"를 누르고, "WildRydes"를 누릅니다. 아래와 같은 화면이 나오면, 왼쪽 메뉴에서 "앱 클라이언트"를 누르고, "앱 클라이언트 추가"를 누릅니다.

{:refdef: style="text-align: center;"}
![07_app_client_add](/assets/img/aws_cognito/07_app_client_add.png) 
{: refdef}

아래와 같이 앱 클라이언트 이름을 입력합니다. 적당히 아무 이름이나 줘도 상관 없지만, 저는 WildRydesWebApp 이란 이름을 사용했습니다. 이 때, "클라이언트 보안키 생성"을 해제해 줍니다. 우리가 사용할 예제는 웹앱이고, 현재 브라우저 기반 애플리케이션에는 클라이언트 암호를 사용할 수 없기 때문입니다.

{:refdef: style="text-align: center;"}
![08_app_client_name](/assets/img/aws_cognito/08_app_client_name.png) 
{: refdef}

맨 아래에서 "앱 클라이언트 생성"을 누르면 앱 클라이언트가 추가됩니다.

# 사용자 풀 ID와 앱 클라이언트 ID 설정

[여기](/aws-amplify/)에서 소개한, AWS에서 예제로 제공하는 웹페이지에 사용자 풀 ID와 앱 클라이언트 ID 설정하겠습니다. 

`js/config.js` 파일을 열고, 아래와 같이 `userPoolId`, `userPoolClientId`, `region` 정보를 넣습니다. 
* userPoolId - 위에서 만든 사용자 풀 ID 입니다.
* userPoolClientId - 위에서 만든 앱 클라이언트 ID 입니다.
* region - 서울의 경우, `ap-northeast-2`입니다.

```
$ vi js/config.js
  1 window._config = {
  2     cognito: {
  3         userPoolId: 'ap-northeast-XXXXYYYYZZZZ',
  4         userPoolClientId: 'ABCDmtt07uapthtdigb0q4it5u',
  5         region: 'ap-northeast-2'
  6     },
  7     api: {
  8         invokeUrl: '' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod',
  9     }
 10 };
```

그리고, 아래와 같이 `js/config.js` 파일을 commit하고 push 합니다. AWS Amplify가 자동으로 웹사이트를 다시 빌드하고 배포합니다. [Amplify 콘솔](https://console.aws.amazon.com/amplify/home)에서 이 과정을 확인할 수 있습니다. 

```
$ git add js/config.js
$ git commit -m "set cognito user pool"
[master 271490a] set cognito user pool
 1 file changed, 3 insertions(+), 3 deletions(-)
$ git push
Username for 'https://git-codecommit.ap-northeast-2.amazonaws.com': PowerUser-at-71406862XXXX
Password for 'https://PowerUser-at-714068624710@git-codecommit.ap-northeast-2.amazonaws.com':
오브젝트 나열하는 중: 7, 완료.
오브젝트 개수 세는 중: 100% (7/7), 완료.
Delta compression using up to 8 threads
오브젝트 압축하는 중: 100% (4/4), 완료.
오브젝트 쓰는 중: 100% (4/4), 420 바이트 | 420.00 KiB/s, 완료.
Total 4 (delta 3), reused 0 (delta 0)
To https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/wildrydes-site
   2753bf7..271490a  master -> master
rostude@E5570 /home/rostude/work/wildrydes-site (master) $
```

JavaScript에 익숙하신 분은 `js` 디렉토리에 있는 소스코드를 분석해 보시면 좋습니다. 

# 가입하고 로그인 해보기

웹사이트의 배포가 끝나면 접속해 봅니다. 제 사이트의 주소는 [https://master.d25kpzfuu2oh24.amplifyapp.com/](https://master.d25kpzfuu2oh24.amplifyapp.com/) 입니다만, 여러분의 주소는 이와는 다를 것입니다.

첫 화면에서 "GIDDY UP!" 버튼을 누르면 웹사이트의 /register.html 페이지로 접속됩니다.
{:refdef: style="text-align: center;"}
![10_giddy_up](/assets/img/aws_cognito/10_giddy_up.png) 
{: refdef}

/register.html 페이지는 아래와 같이 생겼습니다. 여기에 자신의 Email을 입력합니다. 그리고, 예제 웹사이트의 로그인에 사용할 암호를 입력합니다. (Email 계정의 암호를 입력하는 것이 아닙니다.)

{:refdef: style="text-align: center;"}
![11_lets_ryde](/assets/img/aws_cognito/11_lets_ryde.png) 
{: refdef}

그 다음, "LET'S RYDE"를 누르면 /verify.html 페이지로 이동합니다. 그와 동시에, 위에서 입력한 Email로 메일이 하나 전송됩니다. 그 메일에는 아래와 같이 verification code가 들어 있습니다.

{:refdef: style="text-align: center;"}
![12_verification_email](/assets/img/aws_cognito/12_verification_email.png) 
{: refdef}

/verify.html 페이지에서 e-mail로 전달된 6자리의 verification code를 입력합니다. 그리고 VERIFY를 누릅니다.

{:refdef: style="text-align: center;"}
![13_verification_code](/assets/img/aws_cognito/13_verification_code.png) 
{: refdef}

Verification code를 제대로 입력했으면, 여러분의 email 주소가 확인되고 /signin.html 페이지가 나옵니다. 이제, 여러분의 email 주소가 검증되었습니다. 

다시 email 주소와 암호를 입력합니다. 그리고 "SIGN IN"을 누릅니다.

{:refdef: style="text-align: center;"}
![14_signin](/assets/img/aws_cognito/14_signin.png) 
{: refdef}

Email 주소와 암호를 제대로 입력했으면 로그인에 성공하고 /ride.html 페이지로 이동합니다. 그리고 아래와 같이 인증에 성공했다는 메시지가 나옵니다.

{:refdef: style="text-align: center;"}
![15_authenticated](/assets/img/aws_cognito/15_authenticated.png) 
{: refdef}

아직 예제 페이지에 로그인이 된 후에 해야할 작업을 구현하지 않은 상태이므로 지금은 이렇게만 나옵니다만, 어쨌든 로그인까지 마친 상태이므로 이 글의 목표는 달성한 셈입니다.

# AWS Console에서 등록된 사용자 확인

이제, 다시 AWS Cognito 웹페이지로 들어가서, 위에서 등록한 유저가 제대로 등록되어 있는지 알아봅시다. [AWS Cognito Console](https://console.aws.amazon.com/cognito/home)로 들어가거나, AWS Console에서 서비스를 클릭한 다음 모바일 서비스 아래에서 Cognito를 선택합니다. "사용자 풀 관리"를 누르고, 위에서 생성한 사용자 풀인 "WildRydes"를 누릅니다. 

왼쪽 메뉴에서 "사용자 및 그룹"을 누릅니다. 아래와 같이, 위에서 추가한 유저가 등록되어 있음을 볼 수 있습니다.

{:refdef: style="text-align: center;"}
![20_userlist](/assets/img/aws_cognito/20_userlist.png) 
{: refdef}

# 요약

* AWS Cognito에서는 사용자 풀과 자격 증명 풀을 만들 수 있습니다.
* 사용자 풀을 만들면 유저의 가입과 로그인을 처리할 수 있습니다.
* [AWS Cognito Console](https://console.aws.amazon.com/cognito/home)에서 가입한 유저를 관리할 수 있습니다.
* 자격 증명 풀을 만들면 다른 AWS 서비스에 접근 권한을 관리할 수 있습니다. (이 글에서는 설명 안함)
