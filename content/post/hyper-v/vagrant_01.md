---
title: "Apache HTTPD 테스트용 Centos7 구성"
date: 2021-03-05T11:22:13+09:00
categories:
- hyper-v
tags:
- hyper-v
- centos7
keywords:
- vagrant
- hyper-v
#thumbnailImage: //example.com/image.jpg
author: "hyunwoo"
---
로컬 PC로 가상머신을 커맨드라인 명령을 통해 진행 할 수 있을까?
<!--more-->
vagrant 를 사용하여, 한 줄 한 줄 명령어를 통해 CentOS7을 설치 한다.

[목표]
- Hyper-V Nat 네트워크 구성을 활용하여
- CentOS7을 Hyper-V에 설치 해 보자

[사전준비]
 - Hyper-V Nat 네트워크 구성
 - Vagrant 설치 및 사용법 숙지

## Vagrant 로 Hyper-v에 Centos7 프로비저닝하기

{{< alert danger no-icon >}}
반드시, Windows PowerShell의 관리자 권한으로 아래 명령을 수행 해야 함
{{< /alert >}}
