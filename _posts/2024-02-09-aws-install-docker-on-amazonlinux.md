---
title: '[AWS] EC2에 Docker 환경 구성하기'
layout: post
author: 
- 꼼꼼한재은씨
date: '2024-02-09 15:00:00 +0000'
tags:
- AWS 
- docker 
- 도커
- docker-compose 
- 도커 컴포즈
- Amazon Linux 2 
---

AWS의 가상 컴퓨팅 서비스인 EC2는 Debian과 Ubuntu, Suse, CentOS, Windows 등 다양한 운영체제를 AMI(Amazon Machine Image) 형태로 지원한다. EC2 환경에 도커를 설치하는 방법을 알아보자. 

## Amazon Linux 2
Amazon Linux는 AWS에서 공식적으로 관리하는 운영체제로, AWS EC2에 최적화된 OS라고 할 수 있다. 2017년에 발표된 Amazon Linux 2 버전은 현재 가장 많이 사용되고 있으며, AWS 클라우드 최적화된 애플리케이션 패키지를 제공하는 독자적인 (extras)저장소, 인스턴스를 재부팅하지 않고도 커널 보안 및 버그를 패치할 수 있는 Kernel Live Patching 기능 등을 지원한다. 

아쉽게도 이 글을 쓰는 시점인 2024년 2월 기준으로 Amazon Linux 2는 2025년 6월 30일에 지원이 종료될 예정이며, 현재 후속 버전으로 Amazon Linux 2023이 릴리즈된 상태이다. AWS는 새 Amazon Linux 버전에 대해 2년마다 출시, 이후 유지 관리를 3년간 지원하는 정책을 적용한다. 자세한 내용은 아래 링크를 참조한다. 

[Amazon Linux 2 지원정책](https://aws.amazon.com/ko/blogs/korea/update-on-amazon-linux-ami-end-of-life/) 


### Setup Overview
전체 설치 과정은 다음과 같다. 자세한 설명을 원하지 않는다면 이어지는 상세한 내용 대신 아래 스크립트를 실행하면 된다. 스크립트의 주요 시퀀스는 다음과 같다. 

1. Amazon Linux 2에서 도커를 설치한다. 
2. 부팅 시 도커 데몬이 자동 실행되도록 처리한다. 
3. 도커 컴포즈를 설치한다. 


```bash 
# Docker 설치 
$ sudo amazon-linux-extras install docker 
$ sudo usermod -aG docker $USER 

# 재부팅 시에 Docker가 자동으로 실행되도록 설정
$ sudo systemctl enable docker.service 
$ sudo systemctl enable containerd.service  

# Docker Compose 설치 
$ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

이어지는 내용에서는 각 시퀀스에 대해 자세히 설명한다.  

#### Amazon Linux 2에서의 도커 설치

```shell 
$ sudo amazon-linux-extras install docker
$ sudo systemctl start docker.service
$ sudo usermod -a -G docker $USER
```

일반적으로 리눅스 운영체제에서 도커 설치는 docker.com 공식 사이트에서 관리하는 패키지 저장소의 것을 사용한다. Amazon Linux도 마찬가지로 이를 사용할 수 있지만, 호환성 등의 측면에서 AWS에서 제공하는 docker 버전을 사용하기를 권장한다.

구문은 크게 복잡하지 않다. 하나씩 살펴보자. 

###### Line 1 : `sudo amazon-linux-extras install docker`

`extras`는 Amazon Linux2를 안전하게 사용하기 위해 검증된 패키지를 관리하는 저장소이다. 일반적인 오픈 패키지 저장소에는 백도어가 심어진 패키지를 교묘히 올려놓는 유저들도 있어, 해당 패키지를 검증하지 않고 사용하면 백도어가 고스란히 서버에 배포되는 셈이다. 반면 extras는 AWS에서 백도어 등의 요소를 스캔하고, 통과된 패키지만 서빙하므로 안전하다.  

또 시스템 관리자라면 단순히 NGINX를 설치한다고 해도, 어떤 버전을 설치하는 게 전체 시스템에 도움이 될까를 고민하기 마련이다. 이럴 땐 그냥 extras를 이용하면 된다. `amazon-linux-extras` 명령을 사용한다. 


###### Line 2 : `sudo systemctl start docker.service` 

설치된 도커를 데몬으로 실행해 주는 구문이다. systemd 데몬을 사용하여 구동하며, 아래와 같이 `systemctl` 대신 `service`를 사용하여 실행해도 된다. 내부적으로 `/bin/systemctl start docker.service`로 리다이렉트 되므로, 실행 결과는 동일하다.

``` 
# 이 구문은 sudo systemctl start docker.service로 리다이렉트된다. 
sudo service docker start
``` 

이제 `docker ps`를 실행하면 아래와 같이 도커 프로세스 리스트가 출력될 것이다. 물론 아직 실행 중인 도커 컨테이너가 없으므로 리스트는 비어 있다. 
```
$ sudo docker ps  

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

###### Line 3 : `sudo usermod -aG docker $USER`

`docker` 그룹에 현재의 유저를 추가하겠다는 명령이다. sudo를 하지 않고도 도커 컨테이너를 관리 할 수 있다.

`usermod`는 리눅스 사용자 어카운트 정보를 편집하기 위해서 사용한다.

* a : 유저를 추가
* G : 그룹을 편집

해당 명령을 실행하고 나면 **터미널을 재시작해 주어야** 변경된 권한이 반영된다. 반영되었는지 확인할 때에는 아래 명령을 사용한다. 
실행 결과의 맨 끝에 'docker'가 포함되어 있는지 확인한다.

```shell 
 id -nG | grep docker
```

#### systemd를 사용하여 도커 실행 및 부팅 시 도커 데몬이 자동 실행되도록 처리

```bash
$ sudo systemctl enable docker.service
$ sudo systemctl enable containerd.service
```

부팅 시 도커와 ContainerD를 자동으로 시작해주는 명령이다. 

최신 Linux 배포판은 systemd를 사용하여 시스템 부팅 시 시작되는 서비스를 관리한다. 
Debian과 Ubuntu에서는 기본적으로 부팅 시 자동으로 Docker 서비스가 시작되지만, systemd를 사용하는 다른 Linux 배포판에서는 위 구문과 같이 해당 과정을 직접 설정해주어야 한다. 

실행 결과는 아래와 같거나 비슷하다. 

```bash 
$ sudo systemctl enable docker.service
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

$ sudo systemctl enable containerd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /usr/lib/systemd/system/containerd.service.
```

만약 더이상 도커가 자동 실행되지 않도록 하고 싶다면 아래와 같이 `disable` 서브 커맨드를 사용하면 된다. 

```bash 
$ sudo systemctl disable docker.service
$ sudo systemctl disable containerd.service
```

도커 데몬이 제대로 설치되었는지 알아보자. 알려진 간단한 방법 중 하나는 hello-world 앱을 실행해 보는 것이다. 

```bash
$ docker run --rm hello-world 
```

#### docker-compose 설치

```bash
$ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

도커 컴포즈(Docker Compose)는 멀티 컨테이너로 구성된 애플리케이션을 도커 환경에 정의하고 실행하는 도구로, YAML 파일을 사용하여 서비스, 네트워크, 볼륨 등을 설정한다. 
단일 명령어로 여러 컨테이너를 동시에 관리하고 환경을 쉽게 배포할 수 있을 뿐만 아니라 개발, 테스트, 프로덕션 환경에서 일관된 배포를 지원하여 개발자들이 애플리케이션을 쉽게 관리할 수 있게 한다.

`docker run` 명령과 Kubernetes의 중간 규모 애플리케이션을 배포하는 도구라고 생각하면 이해가 쉬울 듯 하다. 

도커 컴포즈는 Amazon Linux Extras에서 관리하지 않는다. 따라서 위와 같이 직접 다운로드하여 설치해야 한다. 설치가 완료되고 나면 `docker-compose -v` 명령을 통해 설치 성공 여부를 확인해볼 수 있다. 

```shell 
$ docker-compose -v
docker-compose version 1.22.0, build f46880fe
```


