---
title:  AWS Amplify - 정적 웹호스팅
tags:   cloud
header:
  teaser: /assets/img/aws_amplify/aws_amplify_teaser.png
  image: /assets/img/aws_amplify/aws_amplify.png
date:   2021-04-18 10:00:00 +0900
sidebar:
  - nav: docs
---

AWS Amplify는 정적 웹호스팅 서비스 입니다. Github pages와 비슷하게, AWS Amplify는 특정 repository에 올려져 있는 html/css/js 코드를 웹서버로 가져와서 서비스하는 방식입니다. 

사용 요금이 부과되고, 12개월의 프리 티어 기간에는 다음과 같은 용량이 무료로 제공됩니다. 다른 서비스와 마찬가지로, 프리 티어 기간에 초보자가 연습삼아 하는 정도로는 추가 요금이 들지 않으니, 걱정하지 않아도 됩니다. 

|    | 정상 요금 | 프리 티어 (무료) |
|----|----------|-----------------|
| 빌드 | 1분당 0.1 USD | 월별 1000분까지 무료 |
| 저장 용량 | 월별 저장된 GB당 0.023 USB | 월별 5GB 저장 |
| 제공 (다른 유저가 읽어간 양) | 제공된 GB당 0.15 USD | 월별 제공 15GB 제공 |

이 글에서는 AWS Amplify를 이용하여 웹서버를 만들어 보겠습니다. 웹페이지 원본은 AWS CodeCommit을 이용할 것입니다. 하지만, 원본을 GitHub, BitBucket, GitLab 등에 올려놓을 수도 있습니다.

# 사전작업

## 필요 패키지 설치

작업은 리눅스 터미널과 AWS 웹 콘솔에서 할 것입니다. 리눅스에는 `git`과 `awscli`가 설치되어 있어야 합니다.

```bash
$ sudo apt install git awscli
```

## 액세스 키

리눅스 터미널에서 작업하려면 액세스 키가 필요합니다. 아직 액세스 키를 만들지 않으신 분은 당장 [AWS IAM 사용자 만들기](/aws-create-iam-user/)로 가셔서 액세스 키를 발급 받으시고 `~/.aws/credentials` 파일에 넣으시기 바랍니다.

## IAM 콘솔에서 Git credential 생성하기

AWS Amplify은 호스팅할 웹페이지 원본의 위치로 GitHub, BitBucket, GitLab, AWS CodeCommit 등을 지원합니다. 이 글에서는 원본 페이지를 AWS CodeCommit에 넣을 것입니다.

{:refdef: style="text-align: center;"}
![01_user](/assets/img/aws_amplify/01_user.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![02_security](/assets/img/02_security/02_security.png) 
{: refdef}

{:refdef: style="text-align: center;"}
![03_create_key](/assets/img/02_security/03_create_key.png) 
{: refdef}


# AWS CodeCommit에서 리포지토리 생성하기



# 생성한 리포지토리 복제

```
$ git clone https://git-codecommit.us-east1.amazonaws.com/v1/repos/wildrydes-site
‘wildrydes-site'로 복제 중 ...
‘https://git-codecommit.us-east-1.amazonaws.com'의 사용자 이름 : XXXXXXXXXX
’USERID'의 비밀번호 : XXXXXXXXXXXX
경고 : 빈 리포지토리를 복제 한 것 같습니다.
```
# 리포지토리 채우기

```
$ cd ~/work/wildrydes-site
$ aws s3 cp s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website ./ --recursive
download: s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website/apply.html to ./apply.html
download: s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website/css/font.css to css/font.css
download: s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website/css/ride.css to css/ride.css
...
```

`ls` 명령을 쳐 보면 아래와 같이 S3로부터 예제파일을 받은 것을 확인할 수 있습니다.
```
$ ls
.git/       css/      favicon.ico  images/     investors.html  register.html  robots.txt   unicorns.html
apply.html  faq.html  fonts/       index.html  js/             ride.html      signin.html  verify.html
```

이제, 이 파일들을 위에서 만든 리포지토리에 올립니다. 유저네임과 패스워드는 1.1에서 받은 것을 입력합니다.

```
$ git add .
$ git commit -m "initial commit"
$ git push
Username for 'https://git-codecommit.ap-northeast-2.amazonaws.com': PowerUser-at-71406862XXXX
Password for 'https://PowerUser-at-714068624710@git-codecommit.ap-northeast-2.amazonaws.com':
오브젝트 나열하는 중: 95, 완료.
오브젝트 개수 세는 중: 100% (95/95), 완료.
Delta compression using up to 8 threads
오브젝트 압축하는 중: 100% (94/94), 완료.
오브젝트 쓰는 중: 100% (95/95), 9.44 MiB | 9.24 MiB/s, 완료.
Total 95 (delta 2), reused 0 (delta 0)
To https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/wildrydes-site
 * [new branch]      master -> master
```

다음으로 AWS Amplify 콘솔을 사용하여 방금 커밋한 웹 사이트를 배포합니다. 
1. [Amplify Console](https://console.aws.amazon.com/amplify/home)에 들어갑니다.
2. 아래 그림처럼 "GET STARTED" 또는 "시작하기"를 누릅니다.

{:refdef: style="text-align: center;"}
![start_amplify](/assets/img/aws_amplify/start_amplify.png) 
{: refdef}

...

{:refdef: style="text-align: center;"}
![14_repository_branch](/assets/img/aws_amplify/14_repository_branch.png) 
{: refdef}


{:refdef: style="text-align: center;"}
![15_build_conf](/assets/img/aws_amplify/15_build_conf.png) 
{: refdef}



{:refdef: style="text-align: center;"}
![17_verified](/assets/img/aws_amplify/17_verified.png) 
{: refdef}

