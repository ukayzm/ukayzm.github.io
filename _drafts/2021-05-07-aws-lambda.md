---
title:  AWS Lambda - 서버리스 컴퓨팅
tags:   cloud
header:
  teaser: /assets/img/aws_lambda/aws_lambda.png
  image: /assets/img/aws_lambda/aws_lambda.png
date:   2021-05-07 18:00:00 +0900
sidebar:
  - nav: docs
---

전통적인 백엔드는 물리적인 서버에 필요한 소프트웨어를 설치하고 필요한 기능을 구현해서 운영하는 형태였습니다. 요즘은 물리적인 서버와 필요 소프트웨어는 클라우드에서 제공하고, 개발자는 기능만 구현하는 형태인 서버리스 컴퓨팅이 대세가 되고 있습니다. 

서버리스 컴퓨팅의 핵심 기능은 AWS Lambda입니다. AWS Lambda의 핵심은 함수 입니다. Lambda 함수의 역할은, http request를 받아서 적당히 처리하고 http response를 생성하는 것입니다. 

거두절미하고, AWS Lambda 함수로 작성한 Hello, World 코드를 한 번 봅시다. JavaScript로 작성되어 있습니다.

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

딱 보고 감이 오시지 않나요? 컴퓨터 언어의 관점에서 말하자면, AWS Lambda 함수는 event, context, callback 세 개의 변수를 파라미터로 받아서 json response를 리턴하는 함수입니다. 이 글을 작성하는 현재, Lambda 함수는 Java, Node.js, C#, Go 또는 Python으로 작성할 수 있습니다.

대부분의 경우, AWS Lambda 함수는 Amazon S3 버킷, DynamoDB, API Gateway 등, 다른 AWS 서비스들과 연동하여 동작합니다. 이 글에서는 Amazon에서 제공하는 예제를 이용하여 DynamoDB에 값을 write하는 Lambda 함수를 만들어 보고 그 동작을 확인해 보겠습니다. 

Amazon의 예제의 기능은

1. 유저의 위치를 입력받아서
2. 가장 가까운 곳에 위치한 유니콘을 찾아내고
3. 유니콘의 정보를 DynamoDB에 저장하고
4. 유니콘의 정보를 유저에게 돌려주는 것입니다.

AWS Lambda 함수에서 DynamoDB에 저장을 할 것이기 때문에, DynamoDB에 테이블을 만들고 테이블 접근 역할을 IAM에서 만들어야 합니다.

작업 순서는 아래와 같습니다.

1. AWS DynamoDB에 테이블 만들기
2. IAM 역할 만들기
3. AWS Lambda 함수 만들기
4. 함수 동작 테스트하기
5. DynamoDB에서 만들어진 항목 확인
6. AWS Lambda 함수 소스 분석

작업은 AWS 웹 콘솔에서만 할 것입니다. AWS 프리티어라면 추가 요금은 들지 않습니다.

# 사전 작업

* IAM 사용자를 만드셨나요? 아니라면, 먼저 [IAM 사용자 만들기](/aws-create-iam-user/)에서 관리자용 IAM 사용자를 만드시기 바랍니다.

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

이 글에서 만드는 Lambda 함수는 DynamoDB에 write하는 기능을 합니다. 이것을 위해서, DynamoDB에 write할 수 있는 권한을 가진 `IAM 역할`을 만들어서 Lambda 함수에 주어야 합니다. 이제 `IAM 역할`을 만들겠습니다.

[AWS IAM 콘솔](https://console.aws.amazon.com/iam/)에 접속합니다. 왼쪽의 "역할"을 누르고 "역할 만들기"를 누릅니다.

<figure>
    <a href="/assets/img/aws_lambda/10_create_role.png" class="align-center"><img src="/assets/img/aws_lambda/10_create_role.png"></a>
</figure>

역할의 유형으로 "AWS 서비스"를 선택하고, 사용 사례로 "Lambda"를 선택합니다. 왼쪽 그림처럼 일반 사용 사례에서 Lambda를 선택해도 되고, 오른쪽 그림처럼 서비스에서 Lambda를 선택한 다음 사용 사례에서 Lambda를 선택해도 됩니다. 그리고 "다음: 권한"을 누릅니다.

<figure class="half">
    <a href="/assets/img/aws_lambda/11_role_lambda.png" class="align-center"><img src="/assets/img/aws_lambda/11_role_lambda.png"></a>
    <a href="/assets/img/aws_lambda/12_role_lambda2.png" class="align-center"><img src="/assets/img/aws_lambda/12_role_lambda2.png"></a>
</figure>




<figure>
    <a href="/assets/img/aws_lambda/13_policy.png" class="align-center"><img src="/assets/img/aws_lambda/13_policy.png"></a>
</figure>

