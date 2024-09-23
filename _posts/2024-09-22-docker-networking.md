---
title: "[Docker] 컨테이너 외부와 통신하는 도커 네트워킹 방법"
date: 2024-09-22 08:15 +0900
lastmod: 2024-09-23 08:15 +0900
categories: Docker
tags: [Internet, WWW, Network, HTTP]
---

## 인터넷과 www

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 인터넷과 www (world wide web)
</div>

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">컴퓨터 네트워크</span>는 <span style='color:rgb(196,58,26);'>통신 및 데이터 교환을 허용</span>하는 연결된 장치의 시스템이다.  
이러한 네트워크와 또 다른 네트워크 간의 연결을 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">🌐 인터넷(Internet)</span>이라 부른다.  
그리고 인터넷에서 데이터를 주고받을 수 있는 프로토콜(통신규약)을 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">HTTP</span>라 한다.

<span style="padding:1px 5px;margin-right: 3px;border: 1px solid #DEDEDE;border-radius:5px;">HTTP를 기반으로 인터넷 통신망에서 동작</span>하는 서비스 중 하나가 `www`이며, 짧게 웹이라 불린다.  
`www`는 초기에는 웹 브라우저의 이름에 국한되었지만,  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">인터넷을 사용하는 컴퓨터들 간의 정보공간</span>의 의미로 발전했다.

## 도커 네트워킹

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 도커 네트워킹이란 컨테이너 외부로의 통신을 허용하는 것을 말한다
</div>

컨테이너 외부로의 통신에는 여러 형태가 있을 수 있다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(1) 웹으로의 요청</span>이나, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(2) 호스트 머신의 데이터베이스와의 통신</span>,  
그리고 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(3) 컨테이너 간의 통신</span>이 있다.

---

`(1)`  
기본적으로 컨테이너는 별다른 설정없이 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">웹에 요청을 보낼 수 있다.</span>  
즉, 컨테이너 외부로 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">HTTP 요청</span>이 전송될 수 있다.

---

`(2)`  
로컬 호스트 머신의 데이터베이스와 통신할 때는 `🚫 localhost`라는 도메인을 사용하면 오류난다.  
<span style='color:#0072BB;'>도커가 이해할 수 있는 특별한 도메인</span>으로 대체해야 한다. `🐋 host.docker.internal`

`ex. mongodb://localhost:27017/test (x)`  
`ex. mongodb://host.docker.internal:27017/test (ㅇ)`

해당 도메인`host.docker.internal`은 도커에 의해 인식되어 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">호스트 머신의 ip 주소</span>를 참조한다.  
도메인이 필요한 곳 어디에든 사용할 수 있다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ HTTP 요청에도 사용할 수 있다.</span>

`ex. http://host.docker.internal:80/test`

---

`(3)`  
컨테이너는 한가지 주요 작업만 수행하는 것이 강력히 권장되고,  
때문에 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">다중 컨테이너로 애플리케이션을 구축</span>하는 것은 매우 일반적이고,  
컨테이너 간 통신은 매우 빈번하게 일어난다.

다른 컨테이너와 통신하려면 결국 그 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">컨테이너의 ip주소</span>를 알면된다.  
`🐋 docker container inspect CONTAINER_NAME`  
`NetworkSettings.IPAddress`를 찾을 수 있다.  
이렇게 ip주소를 알아내어 소스코드에 하드코딩하면 다른 컨테이너와 통신할 수 있지만,  
해당 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">컨테이너의 ip 주소가 변경될 때마다 매번 새 이미지를 빌드</span>해야 하며, 번거롭다.

도커는 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">여러 컨테이너를 하나의 동일한 네트워크에 밀어넣고, 컨테이너 이름을 통해 컨테이너간 통신을 할 수 있다.</span>  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">같은 네트워크 안에 있는 컨테이너 이름</span>은 ip조회 및 해결작업을 자동으로 수행한다.  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">컨테이너 이름(컨테이너 ip주소 참조)을 도메인 즉, 주소로 사용할 수 있다.</span>

`🐋 docker network create NETWORK_NAME`  
: 도커 네트워크 생성

`🐋 docker network ls`
: 도커 네트워크 리스팅

`🐋 docker run --name mongodb --network NETWORK_NAME IMAGE_NAME`
: 이미지를 기반으로 컨테이너를 실행할 때 `--network` 옵션을 써 <span style='color:#0072BB;'>해당 컨테이너를 특정 네트워크에 밀어넣는다.</span>

`ex. mongodb://mongodb:27017/test`  
해당 도메인을 소스코드에 넣으면 데이터베이스가 존재하는 컨테이너와 통신할 수 있다.

☄️ 같은 네트워크 안에 속한 컨테이너 간의 통신만할 때는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">포트를 publish할 필요가 없다.</span>  
: `-p` 옵션은 컨테이너 네트워크 외부에서 컨테이너의 무언가에 연결할 계획일 경우에만 필요하다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 컨테이너 네트워크 내부에서는 모든 컨테이너가 서로 자유롭게 통신할 수 있다.</span>  
`docker run --name app --network NETWORK_NAME IMAGE_NAME (ㅇ)`

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;"><span style="font-weight:bold;">📕 Docker Networks 지원 Driver</span><br>
Docker Networks는 네트워크 동작에 영향을 미치는 다양한 종류의 드라이버를 지원한다.<br>
디폴트 드라이버는 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">bridge 드라이버</span>이다.<br>
해당 드라이버는 <span style="padding:1px 5px;margin-right: 3px;border: 1px solid #DEDEDE;border-radius:5px;">컨테이너가 동일한 네트워크에 있는 경우</span> <span style='color:rgb(196,58,26);'>이름으로 서로를 찾을 수 있다.</span><br><br>
드라이버는 네트워크 생성 시 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">--driver</span> 옵션을 추가하여 간단히 설정할 수 있다.<br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">ex. docker network create --driver host ..</span><br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">stand alone</span> 컨테이너의 경우, 컨테이너와 호스트 시스템 간의 격리가 제거된다.<br>
즉, <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">localhost</span>를 네트워크로 공유한다.

</div>
