---
title:  AWS Cognito - 로그인 관리
tags:   cloud
header:
  teaser: /assets/img/aws_cognito/cognito.jpg
  image: /assets/img/aws_cognito/cognito.jpg
date:   2021-05-02 10:00:00 +0900
sidebar:
  - nav: docs
---

요즘 대부분의 웹페이지나 모바일앱은 가입을 하고 로그인 기능을 제공하는 곳이 대부분입니다. 손쉽게 사용자 가입, 로그인 및 액세스 제어 기능을 제공하기 위한 서비스로 AWS Cognito가 있습니다. E-mail을 ID로 사용할 수도 있고, Apple, Facebook, Google 등의 ID로 로그인할 수도 있습니다.

# 사전작업


{:refdef: style="text-align: center;"}
![01_pool](/assets/img/aws_cognito/01_pool.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![02_create_pool](/assets/img/aws_cognito/02_create_pool.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![03_name](/assets/img/aws_cognito/03_name.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![04_create_pool](/assets/img/aws_cognito/04_create_pool.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![05_pool_id](/assets/img/aws_cognito/05_pool_id.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![06_pool_created](/assets/img/aws_cognito/06_pool_created.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![07_app_client_add](/assets/img/aws_cognito/07_app_client_add.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![08_app_client_name](/assets/img/aws_cognito/08_app_client_name.png) 
{: refdef}


```
$ vi js/config.js
  1 window._config = {
  2     cognito: {
  3         userPoolId: 'ap-northeast-2_lRPcujGF0',
  4         userPoolClientId: '4t9emtt07uapthtdigb0q4it5u',
  5         region: 'ap-northeast-2'
  6     },
  7     api: {
  8         invokeUrl: '' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod',
  9     }
 10 };
```



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
