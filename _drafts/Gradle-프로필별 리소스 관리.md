---
title: "[스프링] Gradle로 프로필(dev/qa/prod) 관리하기"
layout: post
author: ["꼼꼼한재은씨"]
date: '2020-01-06 11:34:00 +0000'
tags:
- Spring
- Gradle 
- profile
- qa
- dev
- stg
- application.properties


---


프로젝트를 구성할 때 꼭 필요한 것 중 하나는 개발 환경이나 테스트 환경, 서비스 환경 등 배포 환경에 따라 설정값을 달리 하는 것이다. DB 연결을 위한 엔드포인트가 대표적인데, 이 값은 대부분의 프로젝트에서 환경 별로 구분해서 사용된다.(서비스용 DB와 개발, 테스트용 DB를 나누는 경우를 떠올리면 된다)

이런 작업의 핵심은 **자동화**에 있다. 수동으로 일일이 설정값을 변경해서 배포한다면, 언젠가는 (거의 반드시라고 해도 좋을 정도로) 문제가 발생한다. 이를테면 상용 서비스를 배포하는데 개발 환경의 설정값이 적용된다거나, 그 반대의 경우 말이다. 테스트 환경인 줄 알고 아무렇게나 입력했는데 그 것이 실 서비스 환경이었다고 생각해보라(아직 그런 일이 일어나지 않았다 하더라도 '아직' 발생하지 않은 것 뿐이지, 일어나지 않는 것은 결코 아니다).

가장 좋은 방법은 컨테이너가 자동으로 배포 환경을 인식하고 그에 맞는 설정값을 읽어오도록 하는 것이다. 이렇게 해 두면 휴먼 에러로 인해 잘못된 설정값이 배포되는 것을 방지할 수 있다.

환경 별로 설정값이 자동 반영되도록 구성하는 작업을 가리켜 **프로필(Profile) 설정**이라고 한다. 영어식 표현이라 그런지 한국의 초보 개발자에게는 다소 익숙하지 않을 수 있지만, 공식 표현이니 잘 익혀두자.

### 프로필 설정하기

스프링에서 프로필 별로 설정값을 나누는 것은 무척 간단하다.

`resources` 디렉토리 아래에 `application-{프로필}.properties`  형태로 파일 이름을 정의해두면 원하는 대로 프로필에 맞는 설정값을 사용할 수 있다. 예시는 다음과 같다. `resources` 디렉토리 안에 정의된 파일 이름을 잘 살펴보자. 

```shell
<프로젝트 루트>
├── src
│   ├── main
│   │   ├── java
│   │   │   └── <패키지경로>
│   │   │       └── Application.java
│   │   ├── resources
│   │   │   └── application.properties
│   │   │   └── application-local.properties
│   │   │   └── application-dev.properties
│   │   │   └── application-qa.properties
│   │   │   └── application-prod.properties
│   └── test
│       └── <이하생략>
├── build.gradle
├── gradlew
├── gradlew.bat
└── settings.gradle
```

jar 형태로 패키징된 애플리케이션을 구동할 때 `-Dspring.profiles.active=qa` 처럼 프로필 옵션을 추가하면 스프링 애플리케이션은 아래 나열된 순서 대로 프로필을 읽어들인다.

```bash
$ java -Dspring.profiles.active=qa -jar project.jar 
```

STEP 1) `application.properties`를 로딩한다.

STEP 2) `application-qa.properties`를 로딩한다.

`application.properties`는 기본 설정 파일이다. 그 위에 `application-{프로필}.properties` 파일이 덮어 쓰여진다. 룰은 다음과 같다.

1. `application.properties`에 정의된 설정값이 `application-{프로필}.properties` 파일에 없다면, `application.properties`의 설정이 사용된다.
2. `application.properties`에 정의되어 있다 하더라도 `application-{프로필}.properties`에서 설정을 오버라이딩하면, 오버라이딩된 값이 사용된다.
3. 실행시 프로필을 지정하지 않으면 `application.properties` 가 우선 적용된다. 그 파일 내에 `spring.profiles.active` 항목이 설정되어 있다면, 이 값을 읽어서 오버라이딩할 설정 파일을 결정한다. 

```properties
# src/main/resources/application.properties
#
# 아래 설정은 application-stg.properties 파일을 읽어 오버라이딩한다. 
spring.profiles.active = stg 
```

`resources` 디렉토리에 `application.properties` 파일이 존재하지 않으면 기본 설정값도 적용되지 않는다. 설정값 오버라이딩 없이 프로필 별 설정값만 사용하고 싶다면 `application.properties` 파일을 제거하면 된다. 

**참고) 프로필 커스터마이징에 대하여**

프로필이 반드시 `dev` / `qa` / `stg` / `prod` 일 필요는 없다. 원하는 대로 프로필을 정의하고, 설정 파일 또한 그 프로필에 맞는 이름으로 준비하면 된다. 필자의 경우 로컬 개발 환경에서 사용하기 위해 `local` 또는 `sqlpro` 프로필을 만들어 두곤 한다.

### 일반 리소스 파일을 프로필 별로 적용하기

`application.properties` 파일이 아닌 일반 리소스 파일을 프로필에 따라 다르게 적용하고 싶다면 약간의 응용이 필요하다. 가능한 방법 중 하나는 Gradle을 이용하여 프로필을 구분하는 것이다. 다음 디렉토리 구조를 보자.

```shell
<프로젝트 루트>
├── src
│   ├── main
│   │   ├── java
│   │   │   └── <패키지경로>
│   │   │       └── Application.java
│   │   ├── resources
│   │   │   └── application.properties
│   │   │   └── application-local.properties
│   │   │   └── application-dev.properties
│   │   │   └── application-qa.properties
│   │   │   └── application-prod.properties
│   │   └── resources-env
│   │       ├── local
│   │       │   └── newfile.txt
│   │       ├── dev
│   │       │   └── newfile.txt
│   │       ├── qa
│   │       │   └── newfile.txt
│   │       └── prod
│   │           └── newfile.txt
│   └── test
│       └── <이하생략>
├── build.gradle
├── gradlew
├── gradlew.bat
└── settings.gradle
```

기존에 사용되던 `resources` 디렉토리는 그대로 두고, 새로이 `resources-env` 를 추가했다. 그리고 그 아래에 프로필 별로 디렉토리를 만들었다. `/resources`에는 공통으로 사용될 설정이, `resources-env/{프로필}`에는 프로필별 파일이 위치한다.

build.gradle은 다음과 같이 수정하자. 이 스크립트는 프로필을 읽고, `resources-env/{프로필}` 디렉토리에 있는 파일을 리소스로 복사한다.

```groovy
ext.profile = (!project.hasProperty('profile') || !profile) ? 'local' : profile;

sourceSets {
    main {
        resources {
            srcDirs += [
                'src/main/resources'
                'src/main/resources-env/${profile}'
            ]
        }
    }
}
```

만약 `profile` =`'local'`이라면 dev / qa / prod 디렉토리에 있는 파일은 빌드에 포함되지 않는다. 따라서 괜히 엉뚱한 환경의 설정 파일이 빌드에 포함되는 찝찝함(?)을 차단할 수 있다.