---
title: "[Issue__mapstruct] Mapstruct 빌드 시, 구현체가 생성되지 않는 현상 해결"
date: 2024-01-25 18:59 +0900
lastmod: 2024-01-25 18:59 +0900
categories: Issue
tags: [mapstruct, lombok, annotationProcessor, compileOnly, getter, setter]
---

## Issue: Mapstruct 빌드 시, 발생하는 문제

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐂 Issue: Mapstruct 빌드 시, 발생하는 문제
</div>

`No implementation was created for ~ due to having a problem in the erroneous element java.util.ArrayList.`

해당 오류는 보통 `mapstruct`를 사용하여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;"> java의 DTO와 Entity 간 변환을 처리할 때 발생</span>하는데  
위 메세지의 핵심의 `ArrayList`와 관련한 문제로 ~ 를 위한 구현체가 생성되지 않았다는 것이다.

이같은 문제의 원인은 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 mapstruct가 롬복보다 먼저 실행되기 때문일 수 있다.</span>  
Dto와 Entity를 작성한 코드를 보면 롬복이 사용되었는데, 결국 `mapstruct`가 변환을 하기 위해서는 롬복의 `getter/setter`를 이용해야 한다.

하지만 `🐘 build.gradle`에서 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">롬복의 경로</span>가 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">mapstruct의 경로</span>보다 아래에 있으면,  
`mapstruct`는 롬복에 의해 생겨날 `getter/setter` 코드를 이용할 수 없다.

`🐘 build.gradle`

```
dependencies {
    compileOnly 'org.projectlombok:lombok'
    implementation 'org.mapstruct:mapstruct:1.5.3.Final'

    annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'
    annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'
    annotationProcessor 'org.projectlombok:lombok'
}
```

확인해보니 위와 같은 순서로 정렬되어 있었다.  
처음엔 `compileOnly '~:lombok'`을 보고 `implementation '~:mapstruct:1.5.3.Final'` 보다 위에 있으니 문제가 없다고 생각했다.

`🍍 compileOnly`  
 : 컴파일 시점에만 참조되며, 빌드된 실행파일에는 포함되지 않는다.  
 롬복 어노테이션 라이브러리처럼 컴파일 시에만 필요하고 런타임에는 필요하지 않기 때문에  
 롬복은 `compileOnly`로 설정할 수 있다.

## resolve: 롬복 컴파일 순서 변경

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐃 resolve: 롬복 컴파일 순서 변경
</div>

`🐘 build.gradle`에서 `annotationProcessor '~:lombok'`이 `annotationProcessor '~:mapstruct-processor:1.5.3.Final'`보다 아래에 정렬되어 있다는 것을 확인했다.

`🍍 Annotation Processor`  
 : <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">컴파일 단계에서 어노테이션에 정의된 일련의 프로세스를 동작</span>하게 하는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">javac에 내장된 도구</span>이다.  
 즉, 컴파일 단계에서 어노테이션을 읽어 관련 자바 소스코드 및 바이트 코드를 생성한다.

결국, `getter/setter`와 관련된 코드가 자동생성되는 시점은 `compileOnly '~:lombok'`이 실행될 때가 아닌,  
`annotationProcessor '~:lombok'`이 실행될 때이다.

즉, `🐘 build.gradle`에서 `annotationProcessor '~:lombok'`의 순서가 `anootationProcessor '~:mapstruct-processor:1.5.3.Final'`보다 위에 있어야 정상동작할 수 있다.

`🐘 build.gradle`

```
dependencies {
    compileOnly 'org.projectlombok:lombok'
    implementation 'org.mapstruct:mapstruct:1.5.3.Final'

    annotationProcessor 'org.projectlombok:lombok'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'
    annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'
}
```
