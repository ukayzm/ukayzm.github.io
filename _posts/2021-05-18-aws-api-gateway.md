---
title:  AWS API Gateway - HTTP/REST API
tags:   cloud
header:
  teaser: /assets/img/aws_api_gateway/awaws-api-gateway-icons_lambda.png
  image: /assets/img/aws_api_gateway/aws-api-gateway-icon.png
date:   2021-05-18 20:00:00 +0900
sidebar:
  - nav: docs
---

Amazon API Gateway는 백엔드의 HTTP 엔드포인트 역할을 제공하는 서비스 입니다. AWS Amplify가 제공하는 정적인 기능에 추가로, API Gateway는 동적인 서비스를 제공할 수 있습니다. 유저는 API Gateway를 통해서 Lambda 함수를 호출할 수 있습니다. 예를 들어, 유저가 GET /list HTTP request를 보내면 API Gateway가 이것을 받아서 "list"라는 Lambda 함수를 호출하고, 그 결과를 받아서 유저에게 HTTP response를 보내주는 식입니다.

* HTTP 기반, 상태 비저장 클라이언트-서버 통신
* 표준 HTTP 메서드(예: GET, POST, PUT, PATCH, DELETE)
* 클라이언트와 서버 간에 상태를 저장하는 전이중 통신을 지원하는 WebSocket

API Gateway는 크게 세 유형의 API를 제공합니다. 디테일로 들어가면 복잡하지만, 일단은 아래와 같은 특징을 가진다고 알고 갑시다.

* REST API - 다기능, 일반적인 AWS 서비스에 접근
* HTTP API - 가볍고 빠른 API
* WebSocket API - 채팅앱처럼 양방향, 지속적인 연결이 필요한 경우에 사용

이 글에서는 REST API를 만들고, Amazon에서 제공하는 예제 웹페이지로 동작을 확인해 보겠습니다. 

작업 순서는 다음과 같습니다.

1. API 만들기
2. 권한 부여자 (authorizer) 만들기
3. 리소스와 메서드를 만들고 Lambda 함수와 권한 부여자 연결하기
4. API 배포하기
5. API를 호출하도록 웹사이트를 수정하고, 호출 확인

# 사전 작업

이전 글에서 소개한 모든 기능이 이 글에서 사용됩니다. 따라서, 아래 강좌를 모두 마쳐야 합니다.

* [AWS Amplify - 정적 웹 호스팅](/aws-amplify/)
* [AWS Cognito - 로그인 관리](/aws-cognito)
* [AWS Lambda - 서버리스 컴퓨팅](/aws-lambda)

# API 만들기

이제, API를 만들어 보겠습니다.

[AWS 웹 콘솔](https://console.aws.amazon.com)에 로그인 하고, 서비스 > API Gateway로 갑니다. 또는, [AWS API gateway 콘솔](https://console.aws.amazon.com/apigateway/)로 바로 접속합니다. 그리고, `API 생성`을 누르면 아래와 같은 화면이 나옵니다. "REST API"의 `구축`을 누릅니다.

<figure>
    <a href="/assets/img/aws_api_gateway/01_create.png" class="align-center"><img src="/assets/img/aws_api_gateway/01_create.png"></a>
</figure>

연결형 서비스를 사용하지 않을 것이므로, 프로토콜은 `REST`를 선택합니다. 

`새 API`를 생성하고, API 이름을 입력합니다. 

엔드포인트는 API의 사용 범위를 지정하는 것으로, 아래와 같이 세 종류가 있습니다.
* 지역 - 특정 지역 (ap-northeat-2 같은 region) 안에서 사용
* 최적화된 에지 - CloudFront를 사용 (즉, 일반적인 인터넷 상에서 사용)
* 프라이빗 - AWS 내 VPC (Virtual Private Cloud)에서만 접근 가능

우리는 인터넷에서 사용할 것이므로, `최적화된 에지`를 선택합니다.

<figure>
    <a href="/assets/img/aws_api_gateway/02_api_name.png" class="align-center"><img src="/assets/img/aws_api_gateway/02_api_name.png"></a>
</figure>

`API 생성`을 누르면 새 API가 만들어 집니다.

# 권한 부여자 (authorizer)

권한 부여자의 용도는 "등록된 사용자에게만 API의 사용을 허가"하는 것으로, 두 가지 유형이 있습니다.

* Cognito 사용자 풀을 이용
* Lambda 함수를 실행한 결과로 권한 부여의 여부를 결정 (Cognito 외에 다른 방식 사용 가능)

여기서 잠깐, [AWS Cognito - 로그인 관리](/aws-cognito)에 나왔던 아래 그림을 다시 보겠습니다. 

<figure>
    <a href="/assets/img/aws_cognito/scenario-cup-cib2.png" class="align-center"><img src="/assets/img/aws_cognito/scenario-cup-cib2.png"></a>
</figure>

1. 사용자가 로그인을 하면 token을 받습니다.
2. Token을 AWS credential로 바꿉니다.
3. Credential을 이용하여 AWS 서비스를 이용합니다.

이 세 단계 중에서 권한부여자는 2 단계를 수행합니다. Token에 대한 자세한 설명은 [이 글](https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)을 참고하시면 좋습니다.

이 글에서는 이전 강좌 [AWS Cognito - 로그인 관리](/aws-cognito)에서 만든 사용자 풀을 활용하여, 등록된 사용자만 접근할 수 있도록 권한 부여자를 구성하겠습니다.

## Cognito 사용자 풀 권한 부여자 생성

먼저, 권한 부여자를 생성합니다. API 리스트에서 방금 만든 API인 `WildRydes`를 선택합니다. 오른쪽 메뉴에서 `권한 부여자`를 누르고, `새로운 권한 부여자 생성`을 누릅니다.

아래와 같이 권한 부여자의 이름을 입력합니다. 유형으로 `Cognito`를 누릅니다. 그리고, 이전 강좌에서 만들었던 사용자 풀 `WildRydes`를 입력합니다. 토큰 원본(Token Source)에 `Authorization`을 입력합니다. 이것은 Authorization 헤더에 토큰이 위치함을 의미합니다. `생성`을 누르면 권한 부여자가 만들어집니다.

<figure>
    <a href="/assets/img/aws_api_gateway/10_create_new_authorizer.png" class="align-center"><img src="/assets/img/aws_api_gateway/10_create_new_authorizer.png"></a>
</figure>

## 브라우저에서 token 복사

이제, 새 브라우저 탭을 엽니다. 그리고, [AWS Amplify - 정적 웹 호스팅](/aws-amplify/)에서 만들었던 웹사이트 도메인의 `/ride.html`로 이동합니다. 저의 경우는 [https://master.d25kpzfuu2oh24.amplifyapp.com/ride.html](https://master.d25kpzfuu2oh24.amplifyapp.com/ride.html) 입니다. 

필요시, 이전 글에서 생성한 사용자로 로그인을 합니다. 그러면 다시 `/ride.html`로 리다이렉션 됩니다.

로그인을 했으므로, token은 받아왔습니다. 하지만 아직 API가 완성되지 않은 상태이므로, 아래와 같이 token 출력 화면이 나옵니다. 

Token을 마우스로 선택하고 Ctrl+C를 눌러 복사합니다.

<figure>
    <a href="/assets/img/aws_api_gateway/11_copy_auth_token.png" class="align-center"><img src="/assets/img/aws_api_gateway/11_copy_auth_token.png"></a>
</figure>

참고로, token은 아래와 같이 긴 문자열 입니다.
```
ABCDaWQiOiI0YlhoXC83bVFxdkdJaHVvczJJNWdSSzgzWSs1TVFaZFg2MmNCMzFOTEs1OD0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZDA1YjAxZC1kODY4LTQ4NWYtYTU0OC1kMzMyY2JiMGU1MzIiLCJhdWQiOiI0dDllbXR0MDd1YXB0aHRkaWdiMHE0aXQ1dSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJldmVudF9pZCI6IjNlODY4YWZmLTJiYTQtNGQ0ZS05YTEwLTg2NzMwZGM1ZGUyYiIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNjIwMjgzMjU3LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAuYXAtbm9ydGhlYXN0LTIuYW1hem9uYXdzLmNvbVwvYXAtbm9ydGhlYXN0LTJfbFJQY3VqR0YwIiwiY29nbml0bzp1c2VybmFtZSI6Imtpam9uZy51aG0tYXQtZ21haWwuY29tIiwiZXhwIjoxNjIwODkxMzUxLCJpYXQiOjE2MjA4ODc3NTEsImVtYWlsIjoia2lqb25nLnVobUBnbWFpbC5jb20ifQ.SjYlzZIIY7ek_3FLdzEP3lSuR9gKFKwnPkKyI0fJpAKpUdHmWuPqUfV4ECANRSXMr6MSUnwgOCB4c4L4WOV3BNIK6bhwMa4GqOj-SOVZWOUbiweqEt5UbXf_ShFt3PriqE4VLaK3i_-ScvSdvLNtwo1TDSrtIYNXrW7T4U_VP_5PznugzTE0_6o6Qs91uJswStzbkH4d3OVpcHjXLW63XrXV1-PTbh4gsKjKowSBhY8woqYEPXN7JGgfiund3erS1YY3aCee2ndy8e0529XC2lcAnXrEaDi64nJ_E3eIhM9ivgtgDjB3tn6GUXn6SY_ldOgfAITtO6ENiSxwQkdHOw
```

## 권한 부여자 테스트

다시 API Gateway 콘솔로 돌아옵니다. 아래와 같이 권한 부여자를 선택하고 `테스트`를 누릅니다.

<figure>
    <a href="/assets/img/aws_api_gateway/12_test.png" class="align-center"><img src="/assets/img/aws_api_gateway/12_test.png"></a>
</figure>

위에서 복사한 token을 붙여 넣고 `테스트`를 누릅니다. Token을 정상적으로 입력했다면 아래와 같이 `응답코드:200`이 출력되고, 사용자에 대한 클레임이 보입니다.

<figure>
    <a href="/assets/img/aws_api_gateway/13_paste_and_test.png" class="align-center"><img src="/assets/img/aws_api_gateway/13_paste_and_test.png"></a>
</figure>

# 새 리소스 및 메서드 생성

API가 만들어졌고, 권한 부여까지 확인 했습니다. 이제 리소스와 메서드를 생성하고, Lambda 함수와 권한 부여자를 연결합니다.

* API에 `/ride`라는 리소스를 생성
* 리소스에 `POST` 메서드를 생성하고 Lambda 함수 연결
* 메서드에 권한 부여자 설정

이렇게 하면, 유저가 POST /ride 요청을 보내면 우리가 만든 Lambda 함수가 호출되도록 할 수 있습니다.

## 리소스 생성

왼쪽 탐색 창에서 WildRydes API 아래 있는 리소스(Resources)를 클릭합니다. 작업(Actions) 드롭다운에서 리소스 생성(Create Resource)을 선택합니다. `ride`를 Resource Name(리소스 이름)으로 입력합니다. 리소스 경로(Resource Path)가 ride로 설정되어 있는지 확인합니다. 리소스에 대해 API Gateway CORS 활성화(Enable API Gateway CORS)를 선택합니다. 리소스 생성(Create Resource)를 누르면 `/ride` 리소스가 만들어 집니다.

<figure>
    <a href="/assets/img/aws_api_gateway/20_create_resource.png" class="align-center"><img src="/assets/img/aws_api_gateway/20_create_resource.png"></a>
</figure>

## 메서드 생성하고 Lambda 함수 연결

새롭게 생성된 /ride 리소스가 선택된 상태로 작업(Action) 드롭다운에서 메서드 생성(Create Method)를 선택합니다. 표시되는 새로운 드롭다운에서 POST를 선택한 다음, 확인 표시를 클릭합니다. 

<figure>
    <a href="/assets/img/aws_api_gateway/21_post.png" class="align-center"><img src="/assets/img/aws_api_gateway/21_post.png"></a>
</figure>

통합 유형에 Lambda Function(Lambda 함수)을 선택합니다. Lambda 프록시 통합 사용(Use Lambda Proxy integration) 상자를 선택합니다. Lambda 리전(Lambda Region)에 사용 중인 리전을 선택합니다. 이전 모듈에서 생성한 함수 이름인 RequestUnicorn을 Lambda 함수(Lambda Function)에 입력합니다. 저장(Save)을 선택합니다. 함수가 존재하지 않는다는 오류가 발생하면 선택한 리전이 이전 모듈에서 사용한 것과 일치하는지 확인합니다.

<figure>
    <a href="/assets/img/aws_api_gateway/22_post_save.png" class="align-center"><img src="/assets/img/aws_api_gateway/22_post_save.png"></a>
</figure>

Lambda 함수를 호출하기 위해 Amazon API Gateway 권한을 부여한다는 메시지가 나타나면 확인(OK)를 선택합니다.

<figure>
    <a href="/assets/img/aws_api_gateway/23_confirm.png" class="align-center"><img src="/assets/img/aws_api_gateway/23_confirm.png"></a>
</figure>

## 메서드에 권한 부여자 설정

방금 만든 `/ride` 리소스의 `POST` 메서드를 선택하고, 메서드 요청(Method Request) 카드를 선택합니다. 

<figure>
    <a href="/assets/img/aws_api_gateway/24_method_request.png" class="align-center"><img src="/assets/img/aws_api_gateway/24_method_request.png"></a>
</figure>

승인(Authorization) 옆의 연필 아이콘을 누릅니다. 드롭다운 목록에서 WildRydes Cognito 사용자 풀을 선택하고 확인 표시 아이콘을 클릭합니다.

<figure>
    <a href="/assets/img/aws_api_gateway/26_user_pool.png" class="align-center"><img src="/assets/img/aws_api_gateway/26_user_pool.png"></a>
</figure>

# API 배포하기

API가 다 만들어졌습니다. 이제, 만들어진 API를 배포합니다.

작업(Actions) 드롭다운 목록에서 API 배포(Deploy API)를 선택합니다.

<figure>
    <a href="/assets/img/aws_api_gateway/30_deploy_api.png" class="align-center"><img src="/assets/img/aws_api_gateway/30_deploy_api.png"></a>
</figure>

배포 스테이지(Deployment stage) 드롭다운 목록에서 새 스테이지(New Stage)를 선택합니다. 스테이지 이름은 `prod`로 입력합니다. 그리고, 배포(Deploy)를 누르면 API가 배포됩니다.

<figure>
    <a href="/assets/img/aws_api_gateway/31_deploy.png" class="align-center"><img src="/assets/img/aws_api_gateway/31_deploy.png"></a>
</figure>

완쪽의 스테이지를 누르고 방금 만든 스테이지인 `prod`를 선택하면 "URL 호출"(Invoke URL)이 나옵니다. 다음 단원에서 사용하기 위해 이 URL을 복사해 둡니다.

<figure>
    <a href="/assets/img/aws_api_gateway/32_invode_url.png" class="align-center"><img src="/assets/img/aws_api_gateway/32_invode_url.png"></a>
</figure>

# 웹사이트 구성 업데이트

[AWS Amplify - 정적 웹 호스팅](/aws-amplify/)에서 만든 웹사이트에서, 위에서 만든 API의 호출 URL을 포함하도록 수정해 봅시다. 

아래와 같이 /js/config.js 파일을 열고, _config.api.invokeUrl 키에 `호출 URL`을 붙여 넣습니다. 

```javascript
$ vi js/config.js
  1 window._config = {
  2     cognito: {
  3         userPoolId: 'ap-northeast-2_lRPcujGF0', // e.g. us-east-2_uXboG5pAb
  4         userPoolClientId: '4t9emtt07uapthtdigb0q4it5u', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
  5         region: 'ap-northeast-2' // e.g. us-east-2
  6     },
  7     api: {
  8         invokeUrl: 'https://305i1fr0i3.execute-api.ap-northeast-2.amazonaws.com/prod'
  9     }
 10 };
```

"호출 URL"은 https로 시작해서 prod로 끝나는 문자열 입니다. 참고로, js/ride.js 코드를 보면 아래와 같이 `_config.api.invokeUrl`이 사용되는 것을 볼 수 있습니다.

```javascript
$ vi js/ride.js
 18     function requestUnicorn(pickupLocation) {
 19         $.ajax({
 20             method: 'POST',
 21             url: _config.api.invokeUrl + '/ride',
 22             headers: {
 23                 Authorization: authToken
 24             },
 25             data: JSON.stringify({
 26                 PickupLocation: {
 27                     Latitude: pickupLocation.latitude,
 28                     Longitude: pickupLocation.longitude
 29                 }
 30             }),
 31             contentType: 'application/json',
 32             success: completeRequest,
 33             error: function ajaxError(jqXHR, textStatus, errorThrown) {
 34                 console.error('Error requesting ride: ', textStatus, ', Details: ', errorThrown);
 35                 console.error('Response: ', jqXHR.responseText);
 36                 alert('An error occured when requesting your unicorn:\n' + jqXHR.responseText);
 37             }
 38         });
 39     }
```

이제 수정된 `js/config.js` 파일을 서버에 적용합니다. 아래와 같이 git push만 하면 자동으로 웹사이트에 배포가 됩니다. 이전에도 언급한 대로, 약간의 시간 (대략 1분 미만)이 걸립니다.

```shell
$ git add .
$ git commit -m "invoceUrl"
[master fab0727] invoceUrl
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git push
Username for 'https://git-codecommit.ap-northeast-2.amazonaws.com': PowerUser-at-AAAABBBBCCCC
Password for 'https://PowerUser-at-714068624710@git-codecommit.ap-northeast-2.amazonaws.com':
오브젝트 나열하는 중: 7, 완료.
오브젝트 개수 세는 중: 100% (7/7), 완료.
Delta compression using up to 8 threads
오브젝트 압축하는 중: 100% (4/4), 완료.
오브젝트 쓰는 중: 100% (4/4), 403 바이트 | 403.00 KiB/s, 완료.
Total 4 (delta 3), reused 0 (delta 0)
To https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/wildrydes-site
   271490a..fab0727  master -> master
$
```

# 구현 확인

웹사이트 도메인의 `/ride.html`을 다시 엽니다. 저의 경우는 [https://master.d25kpzfuu2oh24.amplifyapp.com/ride.html](https://master.d25kpzfuu2oh24.amplifyapp.com/ride.html) 입니다. 이제, 이전과 달리 아래와 같이 지도와 함께 "Request Unicorn" 버튼이 출력됩니다. 지도의 한 부분을 클릭하고 "Request Unicorn" 버튼을 누르면 유니콘이 날아오는 애니메이션이 출력됩니다.

<figure>
    <a href="/assets/img/aws_api_gateway/40_verify.png" class="align-center"><img src="/assets/img/aws_api_gateway/40_verify.png"></a>
</figure>

이전 강좌 [AWS Lambda - 서버리스 컴퓨팅](/aws-lambda)에서 만든 Lambda 함수는 찾아낸 유니콘을 DynamoDB에 기록하도록 만들었죠. DynamoDB에 가 보면 아래 그림과 같이 새로 생성된 유니콘을 볼 수 있습니다.

<figure>
    <a href="/assets/img/aws_api_gateway/41_verify_db.png" class="align-center"><img src="/assets/img/aws_api_gateway/41_verify_db.png"></a>
</figure>

물론 CouldWatch에서도 Lambda 함수의 호출 기록을 확인할 수 있습니다.

# 요약

* AWS API Gateway는 HTTP 엔드포인트 역할을 제공합니다. 
* API Gateway를 통해서 Lambda 함수를 호출할 수 있습니다.
* 권한 부여자를 사용하여 등록된 사용자에게만 API의 사용을 허가할 수 있습니다.
