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

<figure>
    <a href="/assets/img/aws_api_gateway/01_create.png" class="align-center"><img src="/assets/img/aws_api_gateway/01_create.png"></a>
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

