---
title: "Demo - WAS Server Provisioning"
date: 2021-03-13T07:58:57+09:00
draft: true
categories:
- infra
tags:
- Tomcat
- Multi Instance
#thumbnailImage: //example.com/image.jpg
author: "hyunwoo"
---
WAS Server 환경을 구성해 보자~
<!--more-->

Getting Started
------------------------------------------------

Tomcat을 사용한 WAS Server 환경을 구성하는 방법에 대한 안내를 제공 합니다.

다음 사항에 대하여 다룹니다.

- Vargant 및 HyperV로 Server Provisioning
- Ansible playbook 및 Host inventory 파일로 Application Provisioning
- WAS용 OS 계정/그룹 생성, Work 폴더 생성
- Tomcat Multi Instance 구성
- 모니터링 계정과 WAS 관리 계정 분리

준비사항
----------------------------------------------------

[링크 - Apache Tomcat](http://tomcat.apache.org/)

> WAS 그룹계정 생성

```
groupadd -g 2000 wasadm
```

> WAS 계정 생성

```
useradd -g 2000 tomcat
```

> WAS Work 폴더 생성 && Tomcat Download

```
mkdir -p /data/was/ && chown -R tomcat:wasadm /data/was && cd /data/was
wget https://downloads.apache.org/tomcat/tomcat-7/v7.0.108/bin/apache-tomcat-7.0.108.tar.gz
```

> oepnjdk 8 설치

```
yum install java-1.8.0-openjdk
yum install java-1.8.0-openjdk-devel
```

> 환경변수 찾기

```
readlink -f /usr/bin/java

JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64

PATH=$PATH:$JAVA_HOME/bin

CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
```
