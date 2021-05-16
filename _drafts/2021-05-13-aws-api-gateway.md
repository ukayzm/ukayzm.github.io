---
title:  AWS API Gateway - HTTP/REST API
tags:   cloud
header:
  teaser: /assets/img/aws_api_gateway/awaws-api-gateway-icons_lambda.png
  image: /assets/img/aws_api_gateway/aws-api-gateway-icon.png
date:   2021-05-13 10:00:00 +0900
sidebar:
  - nav: docs
---

Amazon API Gateway는 백엔드의 HTTP 엔드포인트 역할을 제공하는 서비스 입니다. AWS Amplify가 정적 웹호스팅을 한다면, API Gateway는 동적 웹호스팅이라고 할까요? 유저는 API Gateway를 통해서 Lambda 함수를 호출할 수 있습니다. 예를 들어, 유저가 GET /list HTTP request를 보내면 API Gateway가 이것을 받아서 "list"라는 Lambda 함수를 호출하고, 그 결과를 받아서 유저에게 HTTP response를 보내주는 식입니다.

* HTTP 기반, 상태 비저장 클라이언트-서버 통신
* 표준 HTTP 메서드(예: GET, POST, PUT, PATCH, DELETE)
* 클라이언트와 서버 간에 상태를 저장하는 전이중 통신을 지원하는 WebSocket

API Gateway는 크게 세 유형의 API를 제공합니다. 디테일로 들어가면 복잡하지만, 일단은 아래와 같은 특징을 가진다고 알고 갑시다.

* REST API - 일반적인 AWS 서비스에 접근
* HTTP API - Lambda에 최적화된 기능 제공
* WebSocket API - 채팅앱처럼 지속적인 연결이 필요한 경우에 사용

이 글에서는 REST API를 만들고, Amazon에서 제공하는 예제 웹페이지로 동작을 확인해 보겠습니다. 



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

# 권한 부여자

권한 부여자의 용도는 "특정 사용자에게만 API의 사용을 허가"하는 것으로, 두 가지 유형이 있습니다.

* Cognito 사용자 풀을 이용
* Lambda 함수를 실행한 결과로 권한 부여의 여부를 결정 (Cognito 외에 다른 방식 사용 가능)

우리는 이전 강좌 [AWS Cognito - 로그인 관리](/aws-cognito)에서 사용자 풀을 만들었습니다. 여기에 등록된 사용자만 접근할 수 있도록 Lambda 권한 부여자를 구성하겠습니다.

## Cognito 사용자 풀 권한 부여자 생성

API 리스트에서 방금 만든 API인 `WildRydes`를 선택합니다. 오른쪽 메뉴에서 `권한 부여자`를 누르고, `새로운 권한 부여자 생성`을 누릅니다.

아래와 같이 권한 부여자의 이름을 입력합니다. 유형으로 `Cognito`를 누릅니다. 그리고, 이전 강좌에서 만들었던 사용자 풀 `WildRydes`를 입력합니다. 토큰 원본(Token Source)에 `Authorization`을 입력합니다. 이것은 Authorization 헤더에 토큰이 위치함을 의미합니다. 그리고 `생성`을 누릅니다.

<figure>
    <a href="/assets/img/aws_api_gateway/10_create_new_authorizer.png" class="align-center"><img src="/assets/img/aws_api_gateway/10_create_new_authorizer.png"></a>
</figure>

## 권한 부여자 구성 확인

이제, 새 브라우저 탭을 엽니다. 그리고, [AWS Amplify - 정적 웹 호스팅](/aws-amplify/)에서 만들었던 웹사이트 도메인의 `/ride.html`로 이동합니다. 필요시, 이전 글에서 생성한 사용자로 로그인을 합니다. 그러면 다시 `/ride.html`로 리다이렉션 됩니다.

아직 API가 완성되지 않은 상태이므로, 아래와 같은 화면이 나올 것입니다. 

<figure>
    <a href="/assets/img/aws_api_gateway/11_copy_auth_token.png" class="align-center"><img src="/assets/img/aws_api_gateway/11_copy_auth_token.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/12_test.png" class="align-center"><img src="/assets/img/aws_api_gateway/12_test.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/13_paste_and_test.png" class="align-center"><img src="/assets/img/aws_api_gateway/13_paste_and_test.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/20_create_resource.png" class="align-center"><img src="/assets/img/aws_api_gateway/20_create_resource.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/21_post.png" class="align-center"><img src="/assets/img/aws_api_gateway/21_post.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/22_post_save.png" class="align-center"><img src="/assets/img/aws_api_gateway/22_post_save.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/23_confirm.png" class="align-center"><img src="/assets/img/aws_api_gateway/23_confirm.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/24_method_request.png" class="align-center"><img src="/assets/img/aws_api_gateway/24_method_request.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/25_authorization.png" class="align-center"><img src="/assets/img/aws_api_gateway/25_authorization.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/26_user_pool.png" class="align-center"><img src="/assets/img/aws_api_gateway/26_user_pool.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/30_deploy_api.png" class="align-center"><img src="/assets/img/aws_api_gateway/30_deploy_api.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/31_deploy.png" class="align-center"><img src="/assets/img/aws_api_gateway/31_deploy.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/32_invode_url.png" class="align-center"><img src="/assets/img/aws_api_gateway/32_invode_url.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/40_verify.png" class="align-center"><img src="/assets/img/aws_api_gateway/40_verify.png"></a>
</figure>

<figure>
    <a href="/assets/img/aws_api_gateway/41_verify_db.png" class="align-center"><img src="/assets/img/aws_api_gateway/41_verify_db.png"></a>
</figure>



```
ABCDaWQiOiI0YlhoXC83bVFxdkdJaHVvczJJNWdSSzgzWSs1TVFaZFg2MmNCMzFOTEs1OD0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZDA1YjAxZC1kODY4LTQ4NWYtYTU0OC1kMzMyY2JiMGU1MzIiLCJhdWQiOiI0dDllbXR0MDd1YXB0aHRkaWdiMHE0aXQ1dSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJldmVudF9pZCI6IjNlODY4YWZmLTJiYTQtNGQ0ZS05YTEwLTg2NzMwZGM1ZGUyYiIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNjIwMjgzMjU3LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAuYXAtbm9ydGhlYXN0LTIuYW1hem9uYXdzLmNvbVwvYXAtbm9ydGhlYXN0LTJfbFJQY3VqR0YwIiwiY29nbml0bzp1c2VybmFtZSI6Imtpam9uZy51aG0tYXQtZ21haWwuY29tIiwiZXhwIjoxNjIwODkxMzUxLCJpYXQiOjE2MjA4ODc3NTEsImVtYWlsIjoia2lqb25nLnVobUBnbWFpbC5jb20ifQ.SjYlzZIIY7ek_3FLdzEP3lSuR9gKFKwnPkKyI0fJpAKpUdHmWuPqUfV4ECANRSXMr6MSUnwgOCB4c4L4WOV3BNIK6bhwMa4GqOj-SOVZWOUbiweqEt5UbXf_ShFt3PriqE4VLaK3i_-ScvSdvLNtwo1TDSrtIYNXrW7T4U_VP_5PznugzTE0_6o6Qs91uJswStzbkH4d3OVpcHjXLW63XrXV1-PTbh4gsKjKowSBhY8woqYEPXN7JGgfiund3erS1YY3aCee2ndy8e0529XC2lcAnXrEaDi64nJ_E3eIhM9ivgtgDjB3tn6GUXn6SY_ldOgfAITtO6ENiSxwQkdHOw
```

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

