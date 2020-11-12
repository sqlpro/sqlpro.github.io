---
title: "맥에 OpenJDK 설치하기"
layout: post
author: ["꼼꼼한재은씨"]
date: '2019-11-23 11:34:00 +0000'
tags:
- java
- openjdk
- java8
---

오라클이 자바를 인수한 후로, 꽤 많은 것이 바뀌어 버렸다. 가장 크게 와 닿는 것이 JDK를 다운로드할 때의 귀찮음인데, 이는 오라클이 과거 버전의 JDK를 내려받기 위해서는 필수적으로 오라클 로그인 및 회원 가입을 거쳐야 하도록 만들어버렸기 때문이다. JAVA 11, JAVA 12, JAVA 13, ... 등 버전을 급속도로 올리는 것도 마음에 들지 않기는 매한가지다.

 ![오라클_로그인_-_통합_인증_Single_Sign_On_](/assets/images/오라클_로그인_-_통합_인증_Single_Sign_On_.png)

(JDK를 다운로드 받기 위해서는 일평생 쓸모라고는 없어 보이는 오라클 계정을 만들어 로그인해야 한다. MySQL도 도커 허브에서 설치 가능한 시대에 오라클 계정 쓸 일이 뭐가 있을까..)

필자는 아직도 안정성을 핑계로 개발시 JAVA 8을 즐겨 사용하기 때문에, 귀찮음을 감수하지 않으려면 OpenJDK를 사용해야 한다. 참고로 OpenJDK는 오라클 JDK의 상업적 정책에 반하는 이들이 모여서 만든 새로운 자바이다.

맥에서 OpenJDK를 설치하는 방법은 다음과 같다. 

```shell
$ brew tap AdoptOpenJDK/openjdk
$ brew cask install adoptopenjdk8 
...
######################################################################## 100.0%
==> Verifying SHA-256 checksum for Cask 'adoptopenjdk8'.
==> Installing Cask adoptopenjdk8
==> Running installer for adoptopenjdk8; your password may be necessary.
==> Package installers may write to any location; options such as --appdir are ignored.
installer: Package name is AdoptOpenJDK
installer: Installing at base path /
installer: The install was successful.
package-id: net.adoptopenjdk.8.jdk
version: 1.0
volume: /
location:
install-time: 1605180582
🍺  adoptopenjdk8 was successfully installed!

$ java -version
openjdk version "1.8.0_275"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_275-b01)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.275-b01, mixed mode)
```

만약 기존에 이미 다른 버전의 JDK가 설치되어 있다면 아래와 같이 삭제한다. 

```shell
$ sudo brew cask uninstall java
```

OpenJDK의 다른 버전을 설치하고 싶다면 아래 목록 중에서 하나를 선택하면 된다. 

- OpenJDK8 - `adoptopenjdk8`
- OpenJDK9 - `adoptopenjdk9`
- OpenJDK10 - `adoptopenjdk10`
- OpenJDK11 - `adoptopenjdk11`

