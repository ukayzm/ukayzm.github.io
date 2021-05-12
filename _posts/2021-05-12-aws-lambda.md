---
title:  AWS Lambda - 서버리스 컴퓨팅
tags:   cloud
header:
  teaser: /assets/img/aws_lambda/aws_lambda.png
  image: /assets/img/aws_lambda/aws_lambda.png
date:   2021-05-12 10:00:00 +0900
sidebar:
  - nav: docs
---

전통적인 백엔드는 물리적인 서버에 필요한 소프트웨어를 설치하고 필요한 기능을 구현해서 운영하는 형태였습니다. 요즘은 물리적인 서버와 필요 소프트웨어를 클라우드에서 제공하는 서버리스 컴퓨팅이 대세가 되고 있습니다. 골치아픈 서버 관리 기능은 클라우드에게 맡기고 개발자는 기능만 구현하면 되기 때문입니다. 

서버리스 컴퓨팅의 핵심 기능은 AWS Lambda입니다. AWS Lambda의 핵심은 Lambda 함수 입니다. Lambda 함수의 역할은, http request를 받아서 적당히 처리하고 http response를 생성하는 것입니다.

# Hello World Lambda 함수

거두절미하고, JavaScript로 작성한 Hello World Lambda 함수의 코드를 한 번 봅시다.

```javascript
exports.handler = (event, context, callback) => {
    const response = {
        statusCode: 200,
        body: JSON.stringify('Hello, World'),
    };
    return response;
};
```

위 코드가 실행되어 리턴하는 결과는 아래와 같습니다. 
```json
{
  "statusCode": 200,
  "body": "\"Hello, World\""
}
```

딱 보고 감이 오시나요? 컴퓨터 언어의 관점에서 말하자면, AWS Lambda 함수는 event, context, callback 세 개의 변수를 파라미터로 받아서, 적당한 작업을 수행하고, 결과로 json response를 리턴하는 함수입니다. Node.js 뿐 아니라, Java, C#, Go 또는 Python으로도 작성할 수 있습니다.

대부분의 경우, AWS Lambda 함수는 Amazon S3 버킷, DynamoDB, API Gateway 등 다른 AWS 서비스들과 연동하여 동작합니다. 

# 들어가며

이 글에서는 Amazon에서 제공하는 예제를 이용하여 DynamoDB에 값을 write하는 Lambda 함수를 만들어 보고 그 동작을 확인해 보겠습니다. 

Amazon의 예제의 기능은

1. 유저의 위치를 입력받아서
2. 가장 가까운 곳에 위치한 유니콘을 찾아내고 (구현을 간단하게 하기 위해서 실제로는 랜덤하게 찾음)
3. 찾아낸 유니콘의 정보를 DynamoDB에 저장하고
4. 그 유니콘의 정보를 유저에게 돌려주는 것입니다.

AWS Lambda 함수에서 DynamoDB에 저장을 할 것이기 때문에, Lambda 함수를 만들기 전에 먼저 DynamoDB에 테이블을 만들고 테이블 접근 역할을 IAM에서 만들 것입니다.

작업 순서는 아래와 같습니다.

1. AWS DynamoDB에 테이블 만들기
2. IAM 역할 만들기
3. AWS Lambda 함수 만들기
4. 함수 동작 테스트하기
5. CloudWatch로 모니터링
6. AWS Lambda 함수 소스 분석

작업은 AWS 웹 콘솔에서만 할 것입니다. AWS 프리티어라면 추가 요금은 들지 않습니다.

# AWS DynamoDB에 테이블 만들기

AWS DynamoDB는 키-값 구조를 지원하는 NoSQL 데이터베이스 서비스 입니다. 여기에 데이터를 저장할 테이블을 만들겠습니다.

[AWS DynamoDB 콘솔](https://console.aws.amazon.com/dynamodb/)에 접속합니다. 관리자 권한이 있는 사용자로 로그인 합니다.

아무 테이블도 만들지 않은 사용자는 아래와 같은 화면이 나옵니다. "테이블 만들기"를 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/01_create_table.png" class="align-center"><img src="/assets/img/aws_lambda/01_create_table.png"></a>
</figure>

다음 화면에서 테이블 이름과 기본 키를 입력합니다. 저는 아래 그림과 같이 입력했습니다. 그리고 "생성"을 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/02_table.png" class="align-center"><img src="/assets/img/aws_lambda/02_table.png"></a>
</figure>

아래와 같이 테이블이 생성 되었습니다. 너무 간단하죠? AWS DynamoDB는 키-값 구조이기 때문에, 테이블 이름과 키 이름 외에는 따로 입력할 것이 없습니다.

<figure>
    <a href="/assets/img/aws_lambda/03_table_spec.png" class="align-center"><img src="/assets/img/aws_lambda/03_table_spec.png"></a>
</figure>

위 그림에서 ARN 필드는 아래에서 사용될 것입니다.

# IAM 역할 만들기

이 글에서 만드는 Lambda 함수는 DynamoDB에 write하는 기능을 합니다. Lambda 함수가 다른 AWS 서비스에 접근하는 것이기 때문에, DynamoDB에 write할 수 있는 권한을 가진 IAM 역할을 만들어서 Lambda 함수에 주어야 합니다.

IAM 역할을 만든다는 것은, 
* 누가 (어떤 서비스가) 그 역할을 수행할 지
* 그 역할로 무슨 작업을 수행할 수 있는지
를 정하는 것입니다. 

[AWS IAM 콘솔](https://console.aws.amazon.com/iam/)에 접속합니다. 왼쪽의 `역할`을 누르고 `역할 만들기`를 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/10_create_role.png" class="align-center"><img src="/assets/img/aws_lambda/10_create_role.png"></a>
</figure>

이 화면에서는 "누가" 이 역할을 수행할 지를 지정합니다. 우리가 지금 만드는 역할은 AWS Lambda 함수가 수행할 역할입니다. 그러므로 역할의 유형으로 `AWS 서비스`를 선택하고, 사용 사례로 `Lambda`를 선택합니다. 왼쪽 그림처럼 일반 사용 사례에서 Lambda를 선택해도 되고, 오른쪽 그림처럼 서비스에서 Lambda를 선택한 다음 사용 사례에서 Lambda를 선택해도 됩니다. 

그리고 "다음: 권한"을 누릅니다.

<figure class="half">
    <a href="/assets/img/aws_lambda/11_role_lambda.png" class="align-center"><img src="/assets/img/aws_lambda/11_role_lambda.png"></a>
    <a href="/assets/img/aws_lambda/12_role_lambda2.png" class="align-center"><img src="/assets/img/aws_lambda/12_role_lambda2.png"></a>
</figure>

이제 역할에 정책을 연결합니다. 정책이란 "무엇"을 할 수 있는지 권한을 정하는 것입니다. 예를 들어, "CloudFront에 로그를 남길 수 있다", "DynamoDB에 write를 할 수 있다" 이런 식이죠.

AWS에는 미리 사전에 정의된 정책이 많이 있습니다. 아래 그림처럼 정책 필터에서 `AWSLambdaBasicExecutionRole`을 검색합니다. 나중에 아시게 되겠지만, 이 정책은 CloudFront에 로그를 남길 수 있는 권한입니다. 체크박스를 선택하고 "다음: 태그"를 누릅니다. 

<figure>
    <a href="/assets/img/aws_lambda/13_policy.png" class="align-center"><img src="/assets/img/aws_lambda/13_policy.png"></a>
</figure>

"태그 추가"는 선택사항 이므로, `다음: 검토`를 눌러 다음으로 넘어가면 아래와 같이 "역할 이름"을 입력하는 화면이 나옵니다. 저는 `WildRydesLambda`라는 이름을 입력했습니다. `역할만들기`를 누르면 역할이 만들어집니다.

<figure>
    <a href="/assets/img/aws_lambda/14_role_name.png" class="align-center"><img src="/assets/img/aws_lambda/14_role_name.png"></a>
</figure>

이제, 서비스 > IAM > 역할을 보면 아래와 같이 만들어진 역할을 확인할 수 있습니다. 역할 이름 `WildRydesLambda`을 눌러, 역할에 부여된 권한을 확인합니다.

<figure>
    <a href="/assets/img/aws_lambda/15_select_role.png" class="align-center"><img src="/assets/img/aws_lambda/15_select_role.png"></a>
</figure>

## 인라인 정책 추가하기

이 역할에는 AWSLambdaBasicExecutionRole 권한이 있습니다. 여기에, DynamoDB에 값을 항목을 추가할 수 있는 권한을 추가해야 합니다. AWS에서 미리 만들어 놓은 정책을 추가해도 되겠지만, 여기에서는 인라인 정책을 추가해 보겠습니다. 인라인 정책이란, 특정 역할에 귀속된 정책입니다. (다른 역할에서는 사용할 수 없다는 뜻입니다.) 

아래 그림의 `인라인 정책 추가`를 눌러, 인라인 정책 추가를 시작합니다.

<figure>
    <a href="/assets/img/aws_lambda/16_add_inline_policy.png" class="align-center"><img src="/assets/img/aws_lambda/16_add_inline_policy.png"></a>
</figure>

`서비스` 오른쪽의 삼각형을 누르고 `DynamoDB`를 검색해서 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/17_select_service.png" class="align-center"><img src="/assets/img/aws_lambda/17_select_service.png"></a>
</figure>

허용되는 작업으로 `putItem`을 선택합니다.

<figure>
    <a href="/assets/img/aws_lambda/18_putitem.png" class="align-center"><img src="/assets/img/aws_lambda/18_putitem.png"></a>
</figure>

리소스를 `특정`으로 선택하고 `ARN 추가`를 누르면, 아래와 같은 창이 나옵니다.

<figure>
    <a href="/assets/img/aws_lambda/19_add_arn.png" class="align-center"><img src="/assets/img/aws_lambda/19_add_arn.png"></a>
</figure>

위에서 만들었던 DynamoDB 테이블의 `ARN`을 입력합니다. 아래 세 개의 필드는 자동으로 채워집니다. Rides 테이블의 ARN이 기억나지 않으시나요? 서비스 > DynamoDB > 테이블 > Rides 에 가면 맨 아래에 나옵니다. 

이제 `추가`를 누릅니다. 그리고, `정책 검토`를 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/20_specify_arn_for_table.png" class="align-center"><img src="/assets/img/aws_lambda/20_specify_arn_for_table.png"></a>
</figure>

아래와 같이 정책의 이름을 입력합니다. 저는 `DynamoDBWriteAccess`라고 입력했습니다. 이제 `정책 생성`을 누르면 인라인 정책이 만들어집니다.

<figure>
    <a href="/assets/img/aws_lambda/21_create_policy.png" class="align-center"><img src="/assets/img/aws_lambda/21_create_policy.png"></a>
</figure>

이제, 서비스 > IAM > 역할 > WildRydesLambda > 권한에 들어가면 아래와 같이 `DynamoDBWriteAccess` 정책이 추가된 것을 볼 수 있습니다.

<figure>
    <a href="/assets/img/aws_lambda/22_policy_added.png" class="align-center"><img src="/assets/img/aws_lambda/22_policy_added.png"></a>
</figure>

# AWS Lambda 함수 만들기

지금까지는 준비 과정이었고, 이제 드디어 본론으로 들어가서 AWS Lambda 함수를 만들어 보겠습니다.

서비스 > Lambda로 들어가거나, [AWS Lambda 콘솔](https://console.aws.amazon.com/lambda/)로 직접 접속합니다. 아래와 같이, 지금은 아무 함수도 없습니다. `함수 생성`을 눌러 함수 만들기를 시작합니다.

<figure>
    <a href="/assets/img/aws_lambda/30_create_function.png" class="align-center"><img src="/assets/img/aws_lambda/30_create_function.png"></a>
</figure>

`함수 이름`을 입력하고, `언어`를 선택합니다.

<figure>
    <a href="/assets/img/aws_lambda/31_function_name.png" class="align-center"><img src="/assets/img/aws_lambda/31_function_name.png"></a>
</figure>

`기본 실행 역할 변경` 옆의 삼각형을 눌러서 Lambda 함수에서 사용할 역할을 부여합니다. 우리는 여기에서 사용할 역할을 이미 위에서 열심히 만들어 두었죠. `기존 역할 사용`을 누르고 `WildRydesLambda` 역할을 선택합니다. 그리고, `함수 생성`을 누르면 함수가 만들어 집니다.

<figure>
    <a href="/assets/img/aws_lambda/32_function_role.png" class="align-center"><img src="/assets/img/aws_lambda/32_function_role.png"></a>
</figure>

## 함수 코드 작성하기

이제, 만들어진 Lambda 함수의 내용을 채워야 합니다. 참고로, 코드 작성과 아래에서 나올 테스트 작업은 개발자가 수도 없이 반복하게 될 일입니다.

<figure>
    <a href="/assets/img/aws_lambda/33_created_function.png" class="align-center"><img src="/assets/img/aws_lambda/33_created_function.png"></a>
</figure>

`코드`를 누르고 `index.js` 파일을 더블클릭 하면, 아래와 같이 default code가 나옵니다.

<figure>
    <a href="/assets/img/aws_lambda/34_edit_function.png" class="align-center"><img src="/assets/img/aws_lambda/34_edit_function.png"></a>
</figure>

Default code는 모두 지웁니다. 그리고, AWS에서 예제로 만든 코드를 넣습니다. 예제 코드는 [requestUnicorn.js](https://webapp.serverlessworkshops.io/serverlessbackend/lambda/requestUnicorn.js){:target="_blank"} 입니다. 예제 코드를 웹브라우저에서 열고 ^C 한 다음 AWS 웹 콘솔에서 붙여넣기를 하면 됩니다.

예제 코드의 내용은 조금 뒤에 분석하기로 하고, 지금은 일단 `deploy`를 눌러 함수 코드 작성을 마칩니다.

<figure>
    <a href="/assets/img/aws_lambda/35_deploy.png" class="align-center"><img src="/assets/img/aws_lambda/35_deploy.png"></a>
</figure>

# 함수 동작 테스트하기

개발자가 신이 아닌 이상, 한번에 완벽한 코드를 만드는 사람은 없습니다. 그래서 코드가 잘 동작하는지 반드시 테스트해 보아야 합니다. 테스트는 특정 이벤트를 발생시켜서 원하는 결과가 나오는지를 보는 과정 입니다. 다시 말하면, 특정 입력을 주고 특정 작업이 수행되고 특정 출력이 나오는지를 확인하는 것입니다.

우리가 만든 Lambda 함수는
* 입력: 사용자의 위치 (여기서는 위도와 경도 값)
* 작업: DynamoDB에 유니콘을 저장
* 출력: 저장된 유니콘을 사용자에게 돌려줌
이런 기능을 수행합니다.

테스트 이벤트는 "특정 입력"에 해당합니다.

## 입력

이제 테스트 이벤트를 작성해 봅시다. `테스트` 옆의 삼각형을 누르고 `Configure test event`를 누릅니다. 

<figure>
    <a href="/assets/img/aws_lambda/40_configure_test.png" class="align-center"><img src="/assets/img/aws_lambda/40_configure_test.png"></a>
</figure>

`새로운 테스트 이벤트 생성`을 누르고, `이벤트 템플릿`으로 "hello-world"를 선택하고, `이벤트 이름`을 부여합니다.

<figure>
    <a href="/assets/img/aws_lambda/41_new_test_event.png" class="align-center"><img src="/assets/img/aws_lambda/41_new_test_event.png"></a>
</figure>

아래 코드를 테스트 코드 창에 붙여 넣고, `생성`을 누릅니다. 

```json
{
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```

<figure>
    <a href="/assets/img/aws_lambda/42_test_event.png" class="align-center"><img src="/assets/img/aws_lambda/42_test_event.png"></a>
</figure>

이제, 테스트 이벤트가 생성되었습니다. `Test` 옆의 삼각형을 누르면 방금 만든 `TestRequestEvent` 가 선택된 것을 볼 수 있습니다.

<figure>
    <a href="/assets/img/aws_lambda/43_test.png" class="align-center"><img src="/assets/img/aws_lambda/43_test.png"></a>
</figure>


## 출력

이제 `Test`를 누르면 테스트가 수행되고 잠시 후 아래와 같이 테스트 결과와 로그가 나옵니다. 아래 그림의 `Response`에 해당하는 빨간 동그라미가 바로 "특정 출력" 입니다.

<figure>
    <a href="/assets/img/aws_lambda/44_test_result.png" class="align-center"><img src="/assets/img/aws_lambda/44_test_result.png"></a>
</figure>

## 작업 결과

서비스 > DynamoDB > 테이블 > Rides > 항목 으로 가 봅니다. 아래와 같이 DB에 항목이 추가되어 있는 것을 볼 수 있습니다. 이 항목은 Lambda 함수의 실행에 의해 추가된 것입니다.

<figure>
    <a href="/assets/img/aws_lambda/45_created_record.png" class="align-center"><img src="/assets/img/aws_lambda/45_created_record.png"></a>
</figure>

# CloudWatch로 모니터링

위에서 IAM 정책에 AWSLambdaBasicExecutionRole를 추가했던 것 기억 하시나요? 이 권한은 CloudWatch에 로그를 남길 수 있는 권한입니다. 이 권한이 있는 Lambda 함수의 호출은 CloudWatch를 통해서 모니터링을 할 수 있습니다. CloudWatch 콘솔로 이동할 필요 없이, Lambda 콘솔 안에서도 모니터링이 가능합니다. 

"코드" 탭에서 `Test` 버튼을 몇 번 누릅니다. 그리고 나서, 아래 그림과 같이 `모니터링`을 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/46_monitoring.png" class="align-center"><img src="/assets/img/aws_lambda/46_monitoring.png"></a>
</figure>

Lambda 함수의 로그가 CloudWatch에 반영될 때 까지는 약간의 시간이 걸립니다. 몇 분 정도 기다리면, 아래와 같이 그래프가 변한 것을 볼 수 있습니다.

<figure>
    <a href="/assets/img/aws_lambda/47_momitoring2.png" class="align-center"><img src="/assets/img/aws_lambda/47_momitoring2.png"></a>
</figure>


# AWS Lambda 함수 소스 분석

서론에서, AWS Lambda 함수는 event, context, callback 세 개의 변수를 파라미터로 받아서, 적당한 작업을 수행하고, 결과로 json response를 리턴하는 함수라고 소개했습니다. 이 관점에서 소스 코드를 분석해 보겠습니다.

```javascript
  6 const AWS = require('aws-sdk');
  7
  8 const ddb = new AWS.DynamoDB.DocumentClient();
```

6째 줄에서 AWS에서 제공하는 aws-sdk 패키지를 사용하고 있습니다. SDK를 사용하면 AWS 서비스를 위한 JavaScript 객체가 제공되어 AWS 서비스에 쉽게 접근할 수 있습니다.

8째 줄에서는 AWS SDK를 이용하여 DynamoDB에 접근하기 위한 객체를 만듭니다.

```javascript
 28 exports.handler = (event, context, callback) => {
```

28째 줄을 보면, `exports.handler` 함수를 정의하고 있죠. Lambda 함수의 진입 지점으로, C의 main함수에 해당하는 부분입니다. 세 개의 parameter가 보이죠?

* event: 사용자가 입력으로 준 값입니다. 우리가 정의한 테스트 이벤트가 여기에 실려서 들어옵니다. 실전에서는 사용자의 HTTP 요청이 들어 있습니다.
* context: 이 객체는 호출, 함수 및 실행 환경에 관한 정보를 가지고 있습니다. 예를 들어, 함수 이름, 버전, ARN, request ID 같은 것들입니다.
* callback: 응답을 전송하기 위해 호출할 수 있는 함수입니다.

```javascript
 86 function findUnicorn(pickupLocation) {
 87     console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
 88     return fleet[Math.floor(Math.random() * fleet.length)];
 89 }
 90
 91 function recordRide(rideId, username, unicorn) {
 92     return ddb.put({
 93         TableName: 'Rides',
 94         Item: {
 95             RideId: rideId,
 96             User: username,
 97             Unicorn: unicorn,
 98             UnicornName: unicorn.Name,
 99             RequestTime: new Date().toISOString(),
100         },
101     }).promise();
102 }
```

findUnicorn()은 유저의 위치로부터 가장 가까이에 있는 유니콘을 찾아내는 함수입니다. 이 예제에서는 최대한 단순하게 하고자, fleet[]에 있는 유니콘을 랜덤하게 리턴하도록 구현되어 있습니다.

recordRide() 함수는 DynamoDB에 유니콘을 저장하는 역할을 합니다. 8째 줄에서 만든 객체를 여기에서 사용합니다.

```javascript
 59         callback(null, {
 60             statusCode: 201,
 61             body: JSON.stringify({
 62                 RideId: rideId,
 63                 Unicorn: unicorn,
 64                 UnicornName: unicorn.Name,
 65                 Eta: '30 seconds',
 66                 Rider: username,
 67             }),
 68             headers: {
 69                 'Access-Control-Allow-Origin': '*',
 70             },
 71         });
```

59째 줄에서 callback 함수를 호출하여 응답을 줍니다. 여기가 "출력"을 담당하는 부분입니다.

# 요약

* 서버리스 컴퓨팅의 핵심 기능은 AWS Lambda이고, AWS Lambda의 핵심은 Lambda 함수 입니다.
* 대부분의 AWS Lambda 함수는 다른 AWS 서비스들과 연동하여 동작합니다. 이를 위해서 Lambda 함수에 IAM 역할과 정책을 부여합니다.
* 언어적인 관점에서 Lambda 함수는 event, context, callback 세 개의 변수를 파라미터로 받아서, 적당한 작업을 수행하고, 결과로 json response를 리턴하는 함수 입니다.
* AWS는 Lambda 함수를 테스트할 수 있는 기능을 제공합니다.
* CloudWatch에서 Lambda 함수의 호출을 모니터할 수 있습니다.
