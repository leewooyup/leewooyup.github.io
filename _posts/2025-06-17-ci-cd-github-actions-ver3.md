---
title: "[CI_CD] (Github Actions) 확장성을 고려한 프로젝트에서 많이 쓰는 CI/CD 구축 방법"
date: 2025-06-17 16:09 +0900
lastmod: 2025-06-22 16:09 +0900
categories: CI_CD
tags:
  [
    Github Actions,
    CodeDeploy,
    codedeploy agent,
    shell script,
    S3,
    EC2,
    role,
    policy,
    tar
  ]
---

## 🪵 AWS CodeDeploy

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #FFD24D;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🪵 서버 (EC2 인스턴스) 가 여러대여도 쉽게 배포를 할 수 있게 도와준다
</div>
<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;color:rgb(196,58,26);">※ 무중단 배포기능도 내재되어 있어 쉽게 무중단 배포를 구현할 수 있다.</span>

`AWS CodeDeploy`가 `EC2`에 명령을 해야하는데, `EC2`가 해당 명령을 알아들을 수 있는 프로그램을 설치해야 한다 `(codedeploy-agent)`

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🧙‍♂️ EC2 內 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">codedeploy-agent</span> 설치</span>

```
sudo apt update && \
sudo apt install -y ruby-full wget && \
cd /home/ubuntu && \
wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install && \
chmod +x ./install && \
sudo ./install auto

systemctl status codedeploy-agent
```

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">active (running)</span> 확인 🔅</span>

```
ls
(install, codedeploy-agent 설치파일)
rm -rf install
(없어도 상관x)
```

## 👹 확장성을 고려한 CI/CD 구축

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	👹 전체적인 흐름
</div>

![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/itd4DAtbYZ.png "Optional title"){: width="600" class="normal"}

`Github Actions`에서 빌드 및 테스트를 수행하고, `S3`에 빌드파일을 전달한다.  
`Github Actions`이 `CodeDeploy`에 배포를 진행하라고 명령하면,  
`CodeDeploy`는 `EC2`에게 `S3`로부터 빌드 파일을 다운받고, 배포를 진행하도록 명령한다.  
(정확히는 `EC2`에 설치된 `codedeploy-agent`에게 명령)

## 🔌 실습 STEP1: 확장성을 고려한 CI/CD 구축, AWS 설정

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 각 리소스 접근할 수 있는 권한 설정 (사용자, 정책, 역할)
</div>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">📔 외부 리소스가 AWS 리소스를 사용할 때 허용할 정책을 가진 사용자를 생성한다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">📔 정책은 특정 리소스에 대한 권한을 지정한다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">※ 하나의 역할은 여러개의 정책을 가질 수 있다. (역할 생성하면서 정책 연결)</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">📔 특정 AWS 리소스가 다른 AWS 리소스에 접근할 권한을 얻을 때 역할을 생성한다.</span>

⛓️ `Github Actions`(외부리소스)에서 `S3`에 접근할 권한 필요 (빌드파일 업로드)  
⛓️ `Github Actions`(외부리소스)에서 `CodeDeploy`에 접근할 권한 필요 (배포 진행 명령)

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>에게 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">IAM</span> 발급 및 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Repository secrets</span>에 저장</span><br>
※ IAM<br><br>
[IAM > 사용자]<br>
🕹️ '사용자 생성'<br>
&emsp;&emsp;사용자 이름: [TEST-SERVER-GITHUB-ACTIONS]<br>
&emsp;&emsp;🕹️ '다음'<br><br>
&emsp;&emsp;권한 옵션<br>
&emsp;&emsp;✅ 직접 정책 연결<br>
&emsp;&emsp;&emsp;✅ AWSCodeDeployFullAccess<br>
&emsp;&emsp;&emsp;✅ AmazonS3FullAccess<br>
&emsp;&emsp;&emsp;🕹️ '다음'<br>
&emsp;&emsp;🕹️ '사용자 생성'<br><br>
생성된 사용자 🖱️<br>
[보안 자격 증명 Tab]<br>
'액세스 키 만들기'<br>
&emsp;&emsp;✅ AWS 외부에서 실행되는 애플리케이션<br>
&emsp;&emsp;🕹️ '다음'<br>
&emsp;&emsp;🕹️ '액세스 키 만들기'<br>
&emsp;&emsp;액세스 키 복사 📑<br>
&emsp;&emsp;비밀 액세스 키 복사 📑<br>
&emsp;&emsp;※ 따로 메모장에 저장<br>
&emsp;&emsp;🕹️ '완료'<br>
&emsp;&emsp;🕹️ '계속'<br><br>
[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">in Github Repository</span>]<br>
[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Settings > Secrets and variables > Actions</span>]<br>
🕹️ '<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">New repository secret</span>'<br>
&emsp;&emsp;<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">AWS_ACCESS_KEY_ID</span><br>
&emsp;&emsp;값 붙여넣기<br>
&emsp;&emsp;🕹️ '<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Add secret</span>'<br><br>
🕹️ '<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">New repository secret</span>'<br>
&emsp;&emsp;<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">AWS_SECRET_ACCESS_KEY</span><br>
&emsp;&emsp;값 붙여넣기<br>
&emsp;&emsp;🕹️ '<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Add secret</span>'

</div>

---

⛓️ `CodeDeploy`에서 `EC2`에 접근할 권한 필요 (배포 진행 명령)

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span>의 S3에 대한 역할 생성</span><br>
[IAM > 역할]<br>
🕹️ '역할 생성'<br>
&emsp;&emsp;✅ AWS 서비스<br>
&emsp;&emsp;서비스 또는 사용 사례: [<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span> 선택]<br>
&emsp;&emsp;'다음'<br><br>
&emsp;&emsp;<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">AWSCodeDeployRole</span><br>
&emsp;&emsp;(필요한 권한들이 이미 다 세팅되어 있다.)<br>
&emsp;&emsp;'다음'<br><br>
&emsp;&emsp;역할 이름: [<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CODE-DEPLOY-ROLE</span>]<br>
&emsp;&emsp;🕹️ '역할 생성'

</div>

---

⛓️ `EC2`에서 `S3`에 접근할 권한 필요 (빌드파일 다운로드)

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 역할에 연결할 정책 생성</span><br>
[IAM > 정책]<br>
🕹️ '정책 생성'<br>
&emsp;&emsp;JSON 🖱️<br>
&emsp;&emsp;아래 json 파일 복붙 📑<br>
&emsp;&emsp;🕹️ '다음'<br><br>

&emsp;&emsp;정책 이름: [<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CODE-DEPLOY-EC2-POLICY</span>]<br>
&emsp;&emsp;🕹️ '정책 생성'<br>

</div>

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": ["s3:Get*", "s3:List*"],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🪄 s3에 접근하고, 조회하는 기능을 허용</span>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 역할 생성 및 EC2에 연결</span><br>
[IAM > 역할]<br>
🕹️ '역할 생성'<br>
&emsp;&emsp;✅ AWS 서비스<br>
&emsp;&emsp;서비스 또는 사용 사례: EC2<br>
&emsp;&emsp;🕹️ '다음'<br><br>

&emsp;&emsp;위에서 생성한 정책 검색 🔍 [CODE-DEPLOY-EC2-POLICY] 및 체크<br>
&emsp;&emsp;🕹️ '다음'<br>

&emsp;&emsp;역할 이름: [CODE-DEPLOY-EC2-ROLE]<br>
&emsp;&emsp;🕹️ '역할 생성'<br><br>

[EC2 > 인스턴스]<br>
✅ [생성했던 인스턴스]<br>
&emsp;&emsp;[작업 > 보안 > IAM 역할 수정]<br>
&emsp;&emsp;[새로 생성한 역할 선택]<br>
&emsp;&emsp;🕹️ 'IAM 역할 업데이트'

</div>

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 빌드파일을 업로드할 S3 생성
</div>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 S3 생성</span><br>
[Amazon S3]<br>
🕹️ '버킷 만들기'<br>
&emsp;&emsp;버킷 이름: [TEST-SERVER-S3]<br>
&emsp;&emsp;🕹️ '버킷 만들기'
</div>

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span> > 애플리케이션 생성
</div>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 애플리케이션 및 배포그룹 생성</span><br>
[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span> > 애플리케이션]<br>
🕹️ '애플리케이션 생성'<br>
&emsp;&emsp;애플리케이션 이름: [<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">TEST-APPLICATION</span>]<br>
&emsp;&emsp;컴퓨팅 플랫폼: ✅ EC2/온프레미스<br>
&emsp;&emsp;🕹️ '애플리케이션 생성'<br><br>

<span style='color:rgb(196,58,26);'>배포그룹: 프로덕션, 스테이징, 디벨롭 환경을 구별하기 위한 단위</span><br>
🕹️ '배포그룹 생성'<br>
&emsp;&emsp;배포 그룹 이름 입력: <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Production</span><br>
&emsp;&emsp;서비스 역할 입력: [IAM에서 생성한 역할 선택]<br><br>

&emsp;&emsp;[애플리케이션 배포 방법 선택]: ✅ 현재 위치<br><br>

&emsp;&emsp;[환경구성]<br>
&emsp;&emsp;✅ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Amazon EC2</span> 인스턴스<br>
&emsp;&emsp;키: Name<br>
&emsp;&emsp;값: [EC2 인스턴스 선택]<br><br>

&emsp;&emsp;[로드밸런서]<br>
&emsp;&emsp;로드 밸런싱 활성화 해제<br><br>

&emsp;&emsp;🕹️ '배포 그룹 생성'

</div>

## 🔌 실습 STEP2: CodeDeploy 설정파일 및 쉘스크립트 파일 작성

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">appspec.yml, CodeDeploy</span> 필수 설정 파일
</div>

해당 yml에는 S3로부터 빌드파일을 다운받고 쉘 스크립트 파일을 실행하라는 내용이 담겨있다.  
<span style='font-size:.98rem;'>※ EC2 내부에 설치된 `codedeploy-agent`가 해당 파일을 읽고 실행한다.</span>  
<span style='color:rgb(196,58,26);font-size:.98rem;'>※ 파일명이 꼭 appspec.yml 여야만 AWS CodeDeploy가 해당 파일을 인식한다.</span>

```yml
version: 0.0
os: linux

# AWS EC2가 S3로부터 어떤 파일을 어느 위치에 다운받을 것인지
files:
  - source: / # S3의 저장한 전체 파일
    destination: /home/ubuntu/test-server

# codedeploy-agent가 작업을 수행할 때 어떤 권한을 가지고 수행할 것인지
permissions:
  - object: /
	owner: ubuntu
	group: ubuntu

# 애플리케이션을 시작할 단계에서 어떤 작업을 수행할 것인지
hooks:
  ApplicationStart:
    - location: scripts/start-server.sh # 쉘스크립트 파일 실행
      timeout: 60
      runas: ubuntu
```

🪄 소스 제공 위치
: `deploy.yml` 파일에서 `aws deploy create-deployment` 명령에서  
`--s3-location bucket=test-server-build-s3,bundleType=tgz,key=$GITHUB_SHA.tar.gz` 옵션을 설정했기 때문에  
`files > source`는 `AWS S3`의 지정한 버킷을 가리킨다.

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">프로젝트 루트 > 📂<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">scripts > start-server.sh</span></span>

```bash
#!/bin/bash

echo "------------------서버 시작------------------"
cd /home/ubuntu/test-server
sudo fuser -k -n tcp 8080 || true
nohup java -jar project.jar > ./output.log 2>&1 &
echo "------------------서버 배포 끝------------------"
```

## 🔌 실습 STEP3: deploy.yml 워크플로우 파일 작성

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">deploy.yml</span>
</div>

```yml
name: Deploy TO EC2 With CodeDeploy

on:
  push:
    branches:
      - main

jobs:
  Deploy:
    runs-on: ubuntu-latest
	steps:
      - name: Github Repository에 업로드한 파일 불러오기
        uses: actions/checkout@v4

      - name: test
        run: |
          ls
          pwd
      - name: JDK 11 ver. 설치
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 11

      - name: Java ver. 확인
        run: java -version

      - name: application.yml 파일 생성
        run: echo "${{ secrets.APPLICATION_PROPERTIES }}" > src/main/resources/application.yml

      - name: 테스트 및 빌드
        run: |
          chmod +x gradlew
          ./gradlew clean build

      - name: 빌드된 파일 이름 변경
        run: mv ./build/libs/*SNAPSHOT.jar ./project.jar

      - name: 압축하기
        run: tar -czvf $GITHUB_SHA.tar.gz project.jar appspec.yml scripts

      - name: AWS Resource에 접근할 수 있게 AWS Credentials 설정
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: ap-northeast-2
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: S3에 프로젝트 폴더 업로드
        run: aws s3 cp --region ap-northeast-2 ./$GITHUB_SHA.tar.gz s3://test-server-build-s3/$GITHUB_SHA.tar.gz

      - name: CodeDeploy를 활용해 EC2에 프로젝트 코드를 배포하라고 명령
        run: aws deploy create-deployment
            --application-name test-server-application
            --deployment-config-name CodeDeployDefault.AllAtOnce
            --deployment-group-name Production
            --s3-location bucket=test-server-build-s3,bundleType=tgz,key=$GITHUB_SHA.tar.gz
```

🪄 압축  
: <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">tar -czvf [압축후 파일명] [압축할 대상1] [압축할 대상2] [압축할 대상1] ...</span>

&nbsp;  
🪄 `Github Actions`에서 AWS 리소스에 접근할 수 있도록 로그인과 유사한 인증이 필요하다.  
: `AWS Credentials`  
🧩 관련 라이브러리: `aws-actions/configure-aws-credentials@v4`

&nbsp;  
🪄 aws 명령어  
: - `aws s3 cp --region ap-northeast-2 p1 s3://~/~.tar.gz`: S3에 있는 파일을 업로드

: - `aws deploy create-deployment`  
<span style='color:rgb(196,58,26);'>배포생성 (개발자 도구 > <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span> > 애플리케이션 > 배포그룹 > '배포 생성' 하는 것과 같다.)</span>  
 `create-deployment` 명령어가 실행되면, 지정한 S3 버킷에서 `.tar.gz` 파일을 다운로드 받는다.  
 해당 압축파일을 임시 디렉토리에 풀고, `appspec.yml`을 찾는다.  
 해당 파일을 파싱하고, 그 내용에 따라 EC2 인스턴스내에서 작업을 수행한다.  
 &nbsp;  
 ※ `CodeDeployDefault.AllAtOnce`: 모든 인스턴스를 한번에 배포하는 옵션 (인스턴스가 여러개 있을 때 중요)

---

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F7F7;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🥊 git push 후 배포 확인</span><br>
[개발자 도구 > <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span> > 배포]<br>
배포 내용이 생겼는 지 확인 🔅<br><br>
[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Amazon S3</span>]<br>
S3에 압축된 파일이 업로드 되었는지 확인 🔅<br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">($GIHUB_SHA.tar.gz)</span><br><br>
[EC2 인스턴스 內]<br>
<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">ls</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">cd test-server</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">ls</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">appspec.yml</span>&emsp;<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">output.log</span>&emsp;<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">project.jar</span>&emsp;<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">scripts</span>&emsp;확인 🔅<br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">lsof -i:8080</span>

</div>

## 🍝 실행된 쉘스크립트 로그 확인

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 codedeploy-agent가 실행시킨 스크립트 파일에 대한 로그를 보고 싶을 때
</div>

```
cd /opt
cd codedeploy-agent
cd deployment-root
cd [START WITH ALPHABET...]
ls
cd [DISTRIBUTION ID]
cd logs
cat scripts.log
```

🪄 배포 ID
: [개발자 도구 > <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">CodeDeploy</span> > 배포]  
해당 배포 id에 해당하는 폴더가 생성되어 있다.
