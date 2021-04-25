---
title:  AWS Amplify - 정적 웹호스팅
tags:   cloud
header:
  teaser: /assets/img/aws_amplify/aws_amplify_teaser.png
  image: /assets/img/aws_amplify/aws_amplify.png
date:   2021-04-25 10:00:00 +0900
sidebar:
  - nav: docs
---

AWS Amplify는 정적 웹호스팅 서비스 입니다. 특정 repository에 올려져 있는 html/css/js 코드를 AWS에서 제공하는 웹서버로 가져와서 서비스하는 방식입니다. 원본 웹페이지는 GitHub, BitBucket, GitLab, AWS CodeCommit 등에 올려놓을 수 있습니다.

다른 AWS 서비스와 마찬가지로 사용 요금이 부과되지만, 12개월의 프리 티어 기간에는 일정 용량이 무료로 제공됩니다. 무료 제공 용량은 초보자가 연습삼아 하는 정도로는 충분하고도 남는 양이니, 추가 요금은 걱정하지 않아도 됩니다. 

|    | 정상 요금 | 프리 티어 (무료) |
|----|----------|-----------------|
| 빌드 | 1분당 0.1 USD | 월별 1000분까지 무료 |
| 저장 용량 | 월별 저장된 GB당 0.023 USB | 월별 5GB 저장 |
| 제공 (다른 유저가 읽어간 양) | 제공된 GB당 0.15 USD | 월별 제공 15GB 제공 |

이 글에서는 AWS Amplify를 이용하여 웹서버를 만들어 보겠습니다. 웹페이지 코드의 위치는 AWS CodeCommit를 이용할 것입니다. 웹페이지를 AWS에서 호스팅하는 것이 목적이기 때문에, 웹페이지를 직접 만들지 않고 AWS에서 예제로 미리 만들어 놓은 것을 가져다 쓰겠습니다. 

# 사전작업

## 필요 패키지 설치

작업은 리눅스 터미널과 AWS 웹 콘솔에서 할 것입니다. 리눅스에는 `git`과 `awscli`가 설치되어 있어야 합니다. 다음 명령으로 설치할 수 있습니다.

```bash
$ sudo apt install git awscli
```

## 액세스 키

리눅스 터미널에서 작업하려면 액세스 키가 필요합니다. 아직 액세스 키를 만들지 않으신 분은 당장 [AWS IAM 사용자 만들기](/aws-create-iam-user/)로 가셔서 액세스 키를 발급 받으시고 `~/.aws/credentials` 파일에 넣으시기 바랍니다.

## IAM 콘솔에서 Git credential 생성하기

AWS Amplify은 호스팅할 웹페이지 원본의 위치로 GitHub, BitBucket, GitLab, AWS CodeCommit 등을 지원합니다. 이 글에서는 원본 페이지를 AWS CodeCommit에 넣을 것입니다. AWS CodeCommit은 5 사용자까지 무료로 사용할 수 있습니다. 용량 제한이 있지만 연습용으로 충분하고도 남으니 추가요금 걱정은 없습니다.

AWS CodeCommit을 이용하려면 git credential을 만들어야 합니다. Git credential은 AWS CodeCommit에 있는 git repository에 접근하기 위한 ID와 암호로, [IAM 대시보드](https://console.aws.amazon.com/iam/)에서 만듭니다. [이 글](/aws-create-iam-user/)에서 개발용으로 만들었던 IAM 사용자인 "PowerUser"를 위한 git credential를 만들겠습니다.

[IAM 대시보드](https://console.aws.amazon.com/iam/)에 접속하여 "Administrator" 사용자로 로그인 합니다. 그리고 아래와 같이 "사용자" > "PowerUser"를 누릅니다. 

{:refdef: style="text-align: center;"}
![01_user](/assets/img/aws_amplify/01_user.png) 
{: refdef}

그리고, "보안 자격 증명" 탭을 누릅니다.

{:refdef: style="text-align: center;"}
![02_security](/assets/img/aws_amplify/02_security.png) 
{: refdef}

아래 쪽으로 스크롤을 하면 "AWS CodeCommit에 대한 HTTPS Git 자격 증명"이 보입니다. 저는 이미 credential을 만든 상태이기 때문에 아래 화면에 credential이 하나 보이네요. "자격 증명 생성"을 누르면 credential을 만들 수 있습니다. 

{:refdef: style="text-align: center;"}
![03_create_key](/assets/img/aws_amplify/03_create_key.png) 
{: refdef}

다음으로, 아래와 같이 자격 증명을 다운로드 받을 수 있는 페이지가 나옵니다. 

{:refdef: style="text-align: center;"}
![035_download_key](/assets/img/aws_amplify/035_download_key.png) 
{: refdef}

위 화면의 "자격 증명 다운로드"를 누르면 .csv 파일을 받을 수 있습니다. .csv 파일 안에는 자격 증명이 들어 있습니다. 

```
$ cat PowerUser_codecommit_credentials.csv 
User Name,Password
PowerUser-at-xxxxyyyyzzzz,ABCDU6I4KMUTVzOBmmlV5XlFftC+Kj1ZnSpMyaE7Hzg=
```

이 자격 증명은 AWS CodeCommit에 파일을 올릴 때 사용합니다.

# AWS CodeCommit에서 리포지토리 생성하기

이제, html/css/js 파일을 넣어 두는 용도로, AWS CodeCommit에 리포지토리를 생성합니다.

먼저, 지역을 "서울"로 바꾸어야 합니다. 그리고 나서, "서비스" > "개발자 도구" > "CodeCommit"을 눌러서 [CodeCommit 대시보드](https://ap-northeast-2.console.aws.amazon.com/codesuite/codecommit/home?region=ap-northeast-2)로 접속합니다. 왼쪽의 "리포지토리"를 누르고, 오른쪽의 "리포지토리 생성"을 누릅니다.

{:refdef: style="text-align: center;"}
![04_create_repository](/assets/img/aws_amplify/04_create_repository.png) 
{: refdef}

아래 화면에서 리포지토리 이름을 줄 수 있습니다. 저는 AWS의 예제와 똑같이 "wildrydes-site"라고 입력했습니다. 그리고, "생성"을 누릅니다.

{:refdef: style="text-align: center;"}
![05_create](/assets/img/aws_amplify/05_create.png) 
{: refdef}

다음으로 연결 단계 화면이 나옵니다. 

{:refdef: style="text-align: center;"}
![06_copy_url](/assets/img/aws_amplify/06_copy_url.png) 
{: refdef}

우리는 사전 작업이서 이미 git을 설치했고, 바로 위에서 PowerUser에게 git 자격 증명도 만들었으므로, 1단계와 2단계는 마친 상태입니다.

위 화면에서 "복사"를 누릅니다. 그리고 리눅스의 터미널에 붙여넣기를 하면 아래와 같이 방금 만든 리포지토리를 리눅스 머신에 복제를 할 수 있습니다.


```
$ cd ~/work
$ git clone https://git-codecommit.us-east1.amazonaws.com/v1/repos/wildrydes-site
‘wildrydes-site'로 복제 중 ...
‘https://git-codecommit.us-east-1.amazonaws.com'의 사용자 이름 : XXXXXXXXXX
’USERID'의 비밀번호 : XXXXXXXXXXXX
경고 : 빈 리포지토리를 복제 한 것 같습니다.
```

리포지토리를 만들기만 했을 뿐 안에 아무 내용도 없는 상태이므로, "빈 리포지토리를 복제 한 것 같습니다."라는 경고가 나오는 것은 당연한 일입니다.

# 리포지토리 채우기

이제 리포지토리에 웹페이지 콘텐츠를 채우고, AWS CodeCommit에 push를 하면 우리가 만든 웹페이지를 인터넷 상에 올릴 수 있습니다.

이 글의 목적은 AWS Amplify의 사용법을 익히는 것이므로, 웹페이지는 Amazon에서 예제로 미리 만들어 놓은 것을 그대로 쓰겠습니다. 아래 명령으로, S3에 올려져 있는 예제를 다운로드 합니다.

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

이제, 이 파일들을 위에서 만든 리포지토리에 올립니다. 유저네임과 패스워드는 앞에서 받은 .csv 파일에 있는 것을 입력합니다.

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

# 웹사이트 배포하기

다음으로 AWS Amplify 콘솔을 사용하여 방금 커밋한 웹 사이트를 배포합니다. 

[Amplify 콘솔](https://console.aws.amazon.com/amplify/home)에 들어갑니다. 아래 그림처럼 "GET STARTED" 또는 "시작하기"를 누릅니다.

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

