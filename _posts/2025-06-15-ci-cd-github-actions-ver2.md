---
title: "[CI_CD] (Github Actions) 서버 밖에서 빌드를 수행하고 빌드된 파일만 전달받는 CI/CD 구축 방법"
date: 2025-06-15 13:50 +0900
lastmod: 2025-06-15 13:50 +0900
categories: CI_CD
tags: [Github Actions, build, library, SCP, step]
---

## 🗿 일반 프로젝트에서 많이 쓰는 CI/CD 구축 방법

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #462679;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🗿 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span> 안에서 빌드와 테스트를 진행하고 빌드된 파일을 전달한다
</div>

AWS EC2에서 빌드하는 것아 아닌 `Github Actions`에서 빌드함으로써  
서버 (EC2) 의 성능에 영향을 거의 주지 않는다.

다만, 무중단 배포 구현이나 여러 EC2 인스턴스에 배포해야하는 상황에서는  
`Github Actions`에서 스크립트를 작성하는 게 꽤 복잡하다.

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">🎯 현업에서 초기 서비스를 구축할 때 많이 활용 (오버 엔지니어링 x)</span>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🍜 EC2 內 필요한 작업</span><br>
해당 방식으로 구축하려면 EC2 내에 JDK만 설치하면 된다.<br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">sudo apt update</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">sudo apt install openjdk-11-jdk -y</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">java -version</span><br>
</div>

## ⚔️ Github Actions 內 사용할 라이브러리

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #2A52BE;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	⚔️ <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Repository</span>에 올린 파일 불러오기, <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">JDK</span> 설치, 파일 전송하기, <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">ssh</span>로 원격 접속
</div>

`🎮 actions/checkout@v4`  
: `Github Repository`에 올린 파일들을 불러올 때 사용

```yml
steps:
  - name: Repository에 올린 파일 불러오기
    uses: actions/checkout@v4
    with: # (선택) 원하는 폴더만 불러오고 싶은 경우 사용
      sparse-checkout: "backend" # 정확한 폴더 경로 지정, 와일드카드 사용 금지
```

`🎮 actions/setup-java@v4`  
: JDK 설치 시 사용

```yml
steps:
  - name: JDK 설치
    uses: actions/setup-java@v4
    with:
      distribution: temurin # 브랜드 선택
      java-version: 11 # 버전 명시
```

`🎮 appleboy/scp-action@v0.1.7`  
: `SCP(Secure Copy)`는 ssh 기반으로 파일을 전송 시 사용 (`Github Actions`에서 빌드된 파일 -> EC2)

```yml
steps:
  - name: SCP로 EC2에 빌드된 파일 전송
    uses: appleboy/scp-action@v0.1.7
    with:
      host: ${{ secrets.EC2_HOST }}
      username: ${{ secrets.EC2_USERNAME }}
      key: ${{ secrets.EC2_PRIVATE_KEY }}
      source: project.jar
      target: /home/ubuntu/test-server/tobe # EC2의 어떤 폴더에 전송된 파일을 저장할 것인가
```

`🎮 appleboy/ssh-action@v1.0.3`  
: ssh를 통해 원격접속 시 사용 (EC2)

```yml
steps:
  - name: ssh로 EC2에 원격 접속
    uses: appleboy/ssh-action@v1.0.3
    with:
      host: ${{ secrets.EC2_HOST }}
      username: ${{ secrets.EC2_USERNAME }}
      key: ${{ secrets.EC2_PRIVATE_KEY }}
      script_stop: true
      script: | # 원격 접속해서 사용할 명령어
        rm -rf /home/ubuntu/test-server/current
        mkdir /home/ubuntu/test-server/current
        mv /home/ubuntu/test-server/tobe/project.jar /home/ubuntu/test-server/current/project.jar
        cd /home/ubuntu/test-server/current
        sudo fuser -k -n tcp 8080 || true
        nohup java -jar project.jar > ./output.log 2>&1 &
        rm -rf /home/ubuntu/test-server/tobe
```

## 🔌 실습: deploy.yml

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 📂<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">.github > 📂workflows > deploy.yml</span>
</div>

```yml
name: Deploy Application To EC2

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
        with:
          sparse-checkout: "backend"

      - name: 불러온 파일 확인 및 위치 파악
        run: |
          ls
          pwd

      - name: JDK 11 ver. 설치
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 11

      - name: 설치한 Java ver. 확인
        run: java -version

      - name: application.yml 파일 생성
        run: |
          cd backend/src/main
          mkdir resources
          cd resources
          pwd
          echo "{% raw %}${{ secrets.APPLICATION_PROPERTIES_AWS }}{% endraw %}" > application-aws.yml
          echo "{% raw %}${{ secrets.APPLICATION_PROPERTIES_JWT }}{% endraw %}" > application-jwt.yml
          echo "{% raw %}${{ secrets.APPLICATION_PROPERTIES }}{% endraw %}" > application.yml

      - name: 테스트 및 빌드
        run: |
          cd backend
          chmod +x gradlew
          ./gradlew clean build

      - name: 빌드된 파일명 변경
        run: |
          cd backend
          mv ./build/libs/*SNAPSHOT.jar ./project.jar

      - name: SCP로 EC2에 빌드된 파일 전송
        uses: appleboy/scp-action@v0.1.7
        with:
          host: {% raw %}${{ secrets.EC2_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.EC2_PRIVATE_KEY }}{% endraw %}
          source: backend/project.jar
          target: /home/ubuntu/test-server/tobe

      - name: ssh로 EC2에 접속
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: {% raw %}${{ secrets.EC2_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.EC2_PRIVATE_KEY }}{% endraw %}
          script_stop: true
          script: |
            rm -rf /home/ubuntu/test-server/current
            mkdir /home/ubuntu/test-server/current
            mv /home/ubuntu/test-server/tobe/backend/project.jar /home/ubuntu/test-server/current/project.jar
            cd /home/ubuntu/test-server/current
            sudo fuser -k -n tcp 8010 || true
            nohup java -jar project.jar > ./output.log 2>&1 &
            rm -rf /home/ubuntu/test-server/tobe
```

### 🐃 내가 헤맨 부분

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F7F7;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🐃 내가 헤맨 부분 - 워크플로우 파일 위치</span><br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Repository</span>에 올린 파일이 다음과 같이 폴더로 하나의 depth가 더 있다.<br>
📂 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">frontend</span><br>
📂 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">backend</span><br>
이 내부에 관련 소스들이 올라가 있는데, 이를 간과하고 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">.github</span> > <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">workflows</span> > <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">deploy.yml</span> 파일을<br>
<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">backend</span> 폴더 內 루트에 생성했는데, 이러면 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>가 인식을 못한다.<br><br>

📂 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">frontend</span><br>
📂 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">backend</span><br>
📂 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">.github</span><br>
이런 식으로 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Repository Root</span>에 워크플로우 파일을 경로에 맞게 작성해줘야 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github Actions</span>에서 인식한다.

</div>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F7F7;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🐃 내가 헤맨 부분 - 각 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">step</span>마다 독립된 셸 세션에서 수행</span><br>
워크플로우 파일에서 <span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">sparse-checkout</span>으로 원하는 폴더만 불러왔을 때(<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">ex. backend</span>)<br>
해당 폴더 내부로 이동해준 뒤 작업해야 한다.<br>
' 불러온 파일 확인 및 위치 파악 '하는 <span style="padding:0 3px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;font-family:var(--bs-font-monospace);font-size:.85rem;">cd backend</span> 명령어로 이동해주면<br>
후속 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">step</span>은 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">backend</span> 폴더 내부에서 작업이 진행될 줄 알았으나<br>
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">각 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">step</span>은 독립된 셸 세션에서 수행되므로, 서로 영향을 받지 않기 때문에</span> 각 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">step</span>에서 폴더 내부로 이동하는 명령어가 필요하다.<br>
</div>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F7F7;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🐃 내가 헤맨 부분 - 워크플로우 파일와 EC2 內 JDK 설치</span><br>
워크플로우 파일 內 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">step</span>에서 JDK를 설치하는 것은 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github</span> 內 임시 가상 서버에서 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">빌드하기 위해서이고</span>,<br>
EC2 內 명령어로 JDK를 설치하는 이유는 전달받은 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">빌드된 파일을 실행시키기 위해서이다.</span><br>
처음에는 워크플로우 파일에서 설치한 JDK를 왜 EC2에 또 설치하는지 헷갈렸는데, 일단은 JDK를 설치하는 컴퓨터가 다르고<br>
각 컴퓨터 (<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Github</span> 內 임시 가상 서버, <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">EC2</span>) 에서 JDK가 필요하기 때문이다.
</div>
