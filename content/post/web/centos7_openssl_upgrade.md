---
title: "Apache HTTPD 테스트용 Centos7 구성 - 2차"
date: 2021-03-08T13:41:42+09:00
categories:
- hyper-v
tags:
- openssl 1.1.1 컴파일
keywords:
- openssl 1.1.1 업그레이드
#thumbnailImage: //example.com/image.jpg
author: "hyunwoo"
---

CentOS 7에 openssl 1.1.1 컴파일 설치 해 보자~
<!--more-->

{{< alert danger no-icon >}}
Apache, Nginx와 같은 웹서버에서 TLS 1.3 프로토콜을 사용하기 위해서 반드시 openssl 업그레이드 필요하며, CentOS 7에서는 openssl 1.1.1g를 컴파일 설치해야 사용 가능 함
{{< /alert >}}


[목표]
- Hyper-V에 설치된 CentOS 7에
- openssl 1.1.1g 컴파일 설치하기

[사전준비]
- [Windows10 Hyper-V Nat Network 구성하기](/2021/03/windows10-hyper-v-nat-network-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/)
- [Apache HTTPD 테스트용 Centos7 구성 - 1차](/2021/03/apache-httpd-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%9A%A9-centos7-%EA%B5%AC%EC%84%B1-1%EC%B0%A8/)

## openssl 기존 버전 확인

```
openssl version
```

## 기존 openssl 삭제

```
yum remove openssl
```

## 기본 라이브러리 설치

```
yum -y install gcc gcc-c++ pcre-devel zlib-devel perl wget
```

## 다운로드 및 config 설정

```
# 다운로드
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar xvfz openssl-1.1.1g.tar.gz
cd openssl-1.1.1g

# config 설정
./config \
--prefix=/usr/local/ssl \
--openssldir=/usr/local/ssl shared zlib

# 컴파일 및 설치
make && make install
```

## openssl-1.1.1g.conf 생성

```
vi /etc/ld.so.conf.d/openssl-1.1.1g.conf

# 파일이 열리면 아래 내용 추가 후 저장
----
/usr/local/ssl/lib
----
```

## shared object (동적 라이브러리 연결) 확인

```
ldconfig -v
```

## 심볼릭링크 걸기

```
# 동적 라이브러리 심볼릭링크
ln -s /usr/local/ssl/lib/libssl.so.1.1 /usr/lib64/libssl.so.1.1
ln -s /usr/local/ssl/lib/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1

# openssl 심볼릭링크
ln -s /usr/local/ssl/bin/openssl /bin/openssl
```

## openssl 버전 확인

```
openssl version

----------------------
OpenSSL 1.1.1g  21 Apr 2020
```

## openssl 지원 프로토콜 확인

```
openssl ciphers -v | awk '{print $2}' | sort | uniq

----------------------
SSLv3
TLSv1
TLSv1.2
TLSv1.3
```

## Ansbie를 사용한 구성 Code 관리

{{< alert info no-icon >}}
[Github 프로젝트 링크](https://github.com/hyunwoo-kr/infra/tree/main/2_openssl_compile_on_centos7)
{{< /alert >}}
