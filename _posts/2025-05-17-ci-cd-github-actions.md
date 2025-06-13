---
title: "[CI_CD] 일련의 로직을 실행시키는 기능인 Github Actions을 이용한 개인 프로젝트 CI/CD 구축 방법"
date: 2025-05-17 14:45 +0900
lastmod: 2025-06-13 15:36 +0900
categories: CI_CD
tags:
  [
    Github Actions,
    Build,
    gradlew,
    test,
    .github,
    workflows,
    Event,
    Job,
    Step,
    AWS EC2,
    SSH,
    yml
  ]
---

## 🪵 CI/CD란

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #FFD24D;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🪵 테스트 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">(Test)</span>, 통합 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">(Merge)</span>, 배포<span style="font-family:var(--bs-font-monospace);font-size:.98rem;">(Deploy)</span>의 과정을 자동화하는 것을 말한다
</div>

새 기능을 구현한 코드가 있을 때마다, 커밋<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">(Commit)</span> 하고 브랜치에 머지<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">(Merge)</span> 해서,  
서버(AWS EC2)에서 직접 매번 업데이트된 코드를 다운받아, 빌드 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">(Build)</span> 하고 테스트 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">(Test)</span> 해서 실행시켜주는 과정이 너무 불편하다.  
이러한 반복적인 배포 과정을 자동화시키기 위해 CI/CD <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">(Continuous Integration, Continuous Deployment)</span>를 배운다.

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F7F7;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🎙️ CI/CD 구축 시, 사용하는 툴</span><br>
<span style="padding:0 6px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span><br>
<span style="padding:0 6px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">Jenkins</span><br>
<span style='color:rgb(196,58,26);'>※ 둘 중 하나마 사용할 줄 알아도 필요한 CI/CD 구성을 전부 할 수 있다.</span><br><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Jenkins</span>의 단점으로는 <span style='color:rgb(196,58,26);'>별도의 서버에 구축</span>해야한다는 점이 있다.<br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>은 별도의 서버 구축 없이 <span style='color:rgb(196,58,26);'><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github</span>에 내장된</span> <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span> 기능을 사용할 수 있기 때문에,<br>
비용적인 측면이나 세팅 측면에서 이점이 있다.
</div>

## 👹 Github Actions란

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	👹 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">Github Actions</span>는 로직을 실행시킬 수 있는 일종의 컴퓨터라고 생각하면 된다
</div>

CI/CD 과정에서 `Github Actions`는 빌드, 테스트, 배포에 대한 로직을 실행시키는 역할을 한다.

```
🧶 동작원리
(1) 코드 작성 후 Commit
(2) Github에 Push
(3) Github이 Push(Event Trigger)라는 이벤트를 감지해서, Github Actions에 작성한 로직이 실행
    a. ssh를 통한 EC2로의 원격접속
    b. git pull
    c. 빌드(build) 및 테스트(Test)
    d. 코드 변경사항 반영 (deploy)
    e. 최신코드로 서버를 재실행
```

`📂.github > 📂workflows > deploy.yml`  
<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;color:rgb(196,58,26);">※ 프로젝트 루트에 .github 디렉토리 아래 workflows 디렉토리 안에 yml 파일을 작성해야 한다.</span>  
<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;color:rgb(196,58,26);">※ yml: 들여쓰기에 따라 인식하는 파일형식</span>

`Github Actions`에서 실행하는 일련의 로직을 `Workflow`라고 한다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(하나의 단위)</span>  
로직이 실행되는 시점 설정을 `Event`라 한다. `(workflow_dispatch, push, pull_request)`

```yml
# Workflow 이름 설정
name: Practice Syntax on Github Actions

on:
  push:
    branches:
      - main

jobs:
  Deploy: # job을 식별하기 위한 id (내 마음대로 고유값 명명)
    runs-on: ubuntu-latest # 운영체제 선택(일종의 컴퓨터), ubuntu 환경 / 가장 최신버전 (latest)

    steps:
      - name: print Hello World
        run: echo "Hello World" # 실제 리눅스 코드

      - name: print Hello world x 2
        run: |
          echo "Hello World"
          echo "Hello World"

      - name: print variables built-in Github Actions
        run: |
          echo $GITHUB_SHA
          echo $GITHUB_REPOSITORY

      - name: print secrets value
        run: |
          echo {% raw %}${{ secrets.MY_NAME }}{% endraw %}
          echo {% raw %}${{ secrets.MY_HOBBY }}{% endraw %}
```

하나의 `Workflow`는 하나 이상의 `Job`으로 구성된다.  
<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;color:rgb(196,58,26);">※ 여러 Job은 기본적으로 병렬적으로 수행된다.</span>  
또한, 하나의 `Job`은 하나 이상의 `Step`으로 구성되어 있다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(Step은 특정 작업을 수행하는 가장 작은 단위이다.)</span>

`$GITHUB_SHA`에는 현재 커밋의 id 값이 담겨있다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(각 커밋은 고유값을 가지고 있다.)</span>  
`$GITHUB_REPOSITORY`에는 현재 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Repository</span> 명</span>이 담겨있다.

`Github Actions`에는 시크릿값을 저장할 수 있는 기능이 있다.  
해당 시크릿값을 조회해도 `***`로 뜬다.

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 시크릿값 저장</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">[⚙️ Settings > 🗝️ Secrets and variables > Actions]</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">🕹️ 'New repository secret'</span>
</div>
  
<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;color:rgb(196,58,26);">※ 한번 키와 값을 만들면 저장된 값을 조회하는 기능이 없다.</span>

## 🗿 개인 프로젝트에서 많이 쓰는 CI/CD 구축 방법

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #462679;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
    🗿 개인 프로젝트에서 많이 쓰는 CI/CD 구축 방법
</div>

🌕 장점

- git pull을 활용해서 프로젝트 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">코드의 변경된 부분만 업데이트하기 때문에, CI/CD 속도가 빠르다.</span>  
  (대부분의 CI/CD 방식은 전체 프로젝트를 통째로 갈아끼우는 방식을 사용한다. 압축해서 전달..)
- CI/CD 툴로 Github Actions만 사용하기 때문에 인프라 구조가 단순하다.

🌑 단점

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">🐖 빌드 작업을 EC2에서 직접 진행하기 때문에 운영하고 있는 서버의 성능에 영향을 미칠 수 있다.</span>  
  <span style='color:rgb(196,58,26);'>(빌드 작업이 생각보다 컴퓨터 자원을 많이 잡아먹는다..)</span>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">🐖 Github 계정정보가 EC2에 저장되기 때문에 협업프로젝트에서는 협업 개발자에게 노출된다.</span>

따라서, 개인프로젝트에서 CI/CD를 심플하고 빠르게 적용시키고 싶을 때 적용한다.

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Spring Boot</span> 애플리케이션 배포</span><br>
[EC2]<br>
✂️ EC2 인스턴스 생성..<br>
(인스턴스 유형: t3a.small)<br>
(※ Spring Boot가 생각보다 무겁기 때문에 이정도 스펙은 필요하다.)<br><br>
[EC2 인스턴스 연결]<br>
[Spring Boot 실행시키기 위한 JDK 설치 필요]<br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">sudo apt update</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">sudo apt install openjdk-11-jdk -y</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">java -version</span><br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git clone [GITHUB_REPOSITORY_PATH]</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">cd [APPLICATION_ROOT_DIRECTORY]</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">chmod +x gradlew</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">./gradlew clean build</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">cd build/libs</span><br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">nohup java -jar [NO_PLAIN_SNAPSHOT].jar &</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">enter ⌨️</span><br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">sudo lsof -i:8080</span><br>
(8080 포트에서 실행되고 있는 프로세스 확인)<br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">sudo fuser -k -n tcp 8080</span><br>
(8080 포트에서 실행되고 있는 프로세스 종료)<br><br>
[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">private Github Repository Authentication</span> 설정 변경]<br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git config --global credential.helper store</span><br>
(한번 계정과 토큰값을 입력하면 그값을 저장해두고, 그 이후로는 물어보지 않는다.)<br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git pull origin main</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">Username for 'https://github.com':</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">Password for 'https://[USER_NAME]@github.com':</span><br>
(설정 한 후 최초에만 입력)<br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">cd ~</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">ls -a</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">cat .git-credentials</span><br>
<span style='color:rgb(196,58,26);'>(여기에 입력했던 계정과 Github 토큰 정보가 노출된다.)</span>

</div>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 CI/CD 구축</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">[in Local Spring Boot Application]</span><br>
프로젝트 루트에 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">📂.github > 📂 workflows > deploy.yml</span> 파일 생성<br><br>
ssh를 통한 원격접속을 할 때, <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>의 라이브러리를 이용한다 ( <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">in MarketPlace</span> )<br>
이 때, 라이브러리에서 정한 규칙을 따르면 된다.<br>
(<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span> (일종의 컴퓨터) -> EC2(컴퓨터), 원격접속)

</div>

```yml
name: Deploy To EC2

on:
  push:
    branches:
      - main

jobs:
  Deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Connect to EC2 with ssh
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: {% raw %}${{ secrets.EC2_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.EC2_PRIVATE_KEY }}{% endraw %}
          script_stop: true
          script: |
            cd /home/ubuntu/[GITHUB_REPOSITORY_NAME]
            git pull origin main
            ./gradlew clean build
            sudo fuser -k -n tcp 8080 || true
            nohup java -jar build/libs/*SNAPSHOT.jar > ./output.log 2>&1 &
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>에 시크릿값 넣기]<br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">[⚙️Settings > 🗝️ Secrets and variables > Actions]</span><br>
🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'New repository secret'</span><br>
&emsp;EC2_HOST<br>
&emsp;[EC2 인스턴스 퍼블릭IP 주소]<br>
&emsp;🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'Add secret'</span><br><br>
🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'New repository secret'</span><br>
&emsp;EC2_USERNAME<br>
&emsp;ubuntu<br>
&emsp;🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'Add secret'</span><br><br>
🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'New repository secret'</span><br>
&emsp;EC2_PRIVATE_KEY<br>
&emsp;[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">keypair (.pem)</span> 값]<br>
&emsp;🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'Add secret'</span><br><br>
keypair 값은 cmd로 .pem 파일이 있는 폴더로 가서<br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">type [KEYPAIR_NAME].pem</span> <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">(in windows)</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">-----BEGIN RSA PRIVATE KEY-----</span> 부터<br><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">-----END RSA PRIVATE KEY-----</span> 까지 복사 📑<br><br>
시크릿값까지 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>에서 설정이 되었다면<br><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git add .</span><br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">git commit -m</span> "커밋메세지"</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git push origin main</span>

</div>

## 🍝 .gitignore에 추가된 application.yml

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 민감한 값이 작성된 application.yml 파일은 .gitignore에 추가되어 github에 올리지 않는다
</div>

`AWS EC2`에서 git pull을 당겨도 `application.yml` 파일이 당연히 없다.  
결국 EC2에서 직접 파일을 생성해준다면, 자동 배포라 할 수 없다.

위에서 SSH 연결에 사용한 `appleboy`에서 환경변수에 대한 규칙을 제공한다.

```yml
name: Deploy To EC2

on:
  push:
    branches:
      - main

jobs:
  Deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Connect to EC2 with ssh
        uses: appleboy/ssh-action@v1.0.3
        # 아래 script 안에서 환경변수를 사용하고 싶다면 먼저 선언
        env:
          APPLICATION_PROPERTIES: {% raw %}${{ secrets.APPLICATION_PROPERTIES }}{% endraw %}
        with:
          host: {% raw %}${{ secrets.EC2_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.EC2_PRIVATE_KEY }}{% endraw %}
          # 위에서 선언한 변수 중 어떤 변수를 쓸지 선택
          envs: APPLICATION_PROPERTIES
          script_stop: true
          script: |
            cd /home/ubuntu/[GITHUB_REPOSITORY_NAME]
            rm -rf src/main/resources/application.yml
            git pull origin main
            echo "$APPLICATION_PROPERTIES" > src/main/resources/application.yml
            ./gradlew clean build
            sudo fuser -k -n tcp 8080 || true
            nohup java -jar build/libs/*SNAPSHOT.jar > ./output.log 2>&1 &
```

다음과 같이, `Marketplace`, `SSH Remote Commands`에서 사용법을 볼 수 있다.

```yml
  - name: Pass environment
    uses: appleboy/ssh-action@v1
+   env:
+     FOO: "BAR"
+     BAR: "FOO"
+     SHA: {% raw %}${{ github.sha }}{% endraw %}
    with:
      host: {% raw %}${{ secrets.HOST }}{% endraw %}
      username: {% raw %}${{ secrets.USERNAME }}{% endraw %}
      key: {% raw %}${{ secrets.KEY }}{% endraw %}
      port: {% raw %}${{ secrets.PORT }}{% endraw %}
+     envs: FOO,BAR,SHA
      script: |
        echo "I am $FOO"
        echo "I am $BAR"
        echo "sha: $SHA"
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span> 에 시크릿값으로 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">application.yml</span> 코드 설정</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">[⚙️Settings > 🗝️ Secrets and variables > Actions]</span><br>
🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'New repository secret'</span><br>
&emsp;APPLICATION_PROPERTIES<br>
&emsp;[<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">application.yml</span> 코드 전체 복붙 📑]<br>
&emsp;🕹️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">'Add secret'</span><br><br>
시크릿값까지 설정 했다면<br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git add .</span><br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">git commit -m</span> "커밋메세지"</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">git push origin main</span><br><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">[In EC2 Instance]</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">[Application Project Root Directory]</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">cd src/main/resources</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">cat applicaiton.yml</span><br>
제대로 반영되었나 확인 🔅<br><br>
단, <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">application.yml</span> 파일의 변경은 수동으로 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github</span>에서 시크릿값을 변경해주는 수 밖에 없다.

</div>

## 🔌 테스트 코드와 배포

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">./gradlew clean build</span> 명령에 테스트까지 포함되어 있다
</div>

`🌱 start.spring.io`에서 SpringBoot Application을 생성하면,  
`📂 src > 📂 test > 📂 java > 📂 [PACKAGE_NAME]` 해당 경로에  
`☕ [APPLICATION_NAME]ApplicationTests.java` 파일이 기본적으로 생성되는데,  
이 파일 내에 있는 테스트 코드를 일부러 터뜨렸을 때, <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>에 작성한 로직 중 `./gradlew clean build` 명령에서 빌드 오류를 뱉고 해당 명령어에서 배포를 중단한다.

```java
@SpringBootTest
class TestServerApplicationTests {

  @Test
  void contextLoads() {
    throw new RuntimeException("Fail"); // 일부로 오류 뱉기
  }
}
```

`./gradlew test` 명령어를 입력해보면 테스트 코드가 실패하는 것을 알 수 있다.  
해당 코드를 github에 push해보고 `Github Actions`의 `Workflow`가 중단된 것을 알 수 있다.

`FAILURE: Build failed with an exception.`

즉, 테스트 코드까지 통과해야 배포가 진행된다.
