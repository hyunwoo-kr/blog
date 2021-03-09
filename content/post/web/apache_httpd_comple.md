---
title: "Apache HTTPD 테스트용 Centos7 구성 - 3차"
date: 2021-03-08T16:41:42+09:00
categories:
- web
tags:
- Apache httpd 2.4.X
- compile
keywords:
- WEB
#thumbnailImage: //example.com/image.jpg
author: "hyunwoo"
---
CentOS 7에 Apache HTTPD 2.4.X 컴파일 설치 해 보자
<!--more-->

[목표]
- Hyper-V에 설치된 CentOS 7에
- Apache HTTPD 2.4.x version을 컴파일 설치하기

[사전준비]
- [Windows10 Hyper-V Nat Network 구성하기](/2021/03/windows10-hyper-v-nat-network-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/)
- [Apache HTTPD 테스트용 Centos7 구성 - 1차](/2021/03/apache-httpd-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%9A%A9-centos7-%EA%B5%AC%EC%84%B1-1%EC%B0%A8/)

# Why? '컴파일'

 - 대부분의 리눅스 운영체제에는 Apache 웹서버가 기본으로 설치되어 있음
 - 하지만, 이것은 오래된 2.2.x version 으로
 - 보안패치도 안 되어 있는 등 실제 현업에서는 사용하지 않습니다.

{{< alert success no-icon>}}
그래서 최신버전의 Apache 웹서버 2.4.x 를 컴파일 하여 설치 해서 사용 합니다.
{{< /alert >}}

## 이전 version의 Apache 웹서버 제거

```
(CentOS) yum remove httpd httpd-*
```

## 빌드 환경설정

컴파일에 필요한 GCC 컴파일러, libtoo,l, make 등을 설치 함

```shell
(CentOS) yum install -y make gcc gcc-c++ autoconf automake libtool pkgconfig findutils
```

## 필요한 Header 파일 설치

```shell
(CentOS) yum install -y zlib-devel openldap-devel pcre-devel openssl-devel libxml2-devel
```

## Apache 웹서버 - 설치준비

 - Apr (Apache Portable Runtime) 설치(컴파일/빌드)
 - expat 설치 (컴파일/빌드)
 - Apr-utils 설치(컴파일/빌드)
 - pcre 설치(컴파일/빌드)

### Apr (Apache Portable Runtime) 설치(컴파일/빌드)

다운로드 주소: https://apr.apache.org/download.cgi
![APR다운로드](/img/web/apache/apr_download.png)

> Apr 1.7.0 다운로드 및 압축풀기

```shell
# HTTP Server 압축 푼 폴더의 srclib로 이동
cd /usr/local/src

# Apr 다운로드
wget https://downloads.apache.org//apr/apr-1.7.0.tar.gz

# 압축풀기 & 폴더이름 변경 (apr-1.7.0 -> apr)
tar xvfz apr-1.7.0.tar.gz && rm -rf apr-1.7.0.tar.gz

cd apr-1.7.0.tar.gz
./configure --prefix=/usr/local/apr
make & make install
```

> 설치 확인

```shell
/usr/local/apr/bin/apr-1-config --version

--------------------------------------------
1.7.0
```

### expat 설치

{{< alert info no-icon >}}
다운로드 주소: https://github.com/libexpat/libexpat/releases
{{< /alert >}}

> 다운로드 및 설치
```
cd /usr/local/src

wget https://github.com/libexpat/libexpat/releases/download/R_2_2_9/expat-2.2.9.tar.gz

tar xvfz expat-2.2.9.tar.gz

cd expat-2.2.9

./configure --prefix=/usr/local/expat

make & make install

```
> 설치 확인

```
/usr/local/expat/bin/xmlwf -v

------------------------------
xmlwf using expat_2.2.9
```


### Apr-util 설치(컴파일/빌드)

{{< alert info no-icon >}}
다운로드 주소: https://apr.apache.org/download.cgi
{{< /alert >}}

![APR_UTIL다운로드](/img/web/apache/apr_util_download.png)

> Apr-util 1.6.1 설치

```shell
# HTTP Server 압축 푼 폴더의 srclib로 이동
cd /usr/local/src

# 다운로드
wget https://downloads.apache.org//apr/apr-util-1.6.1.tar.gz

# 압축풀기 & 폴더이름 변경 (apr-util-1.6.1 -> apr-util)
tar xvfz apr-util-1.6.1.tar.gz && rm -rf apr-util-1.6.1.tar.gz

cd apr-util-1.6.1

./configure --prefix=/usr/local/apr-util \
--with-apr=/usr/local/apr \
--with-expat=/usr/local/expat

make && make install

```
> 설치 확인

```
/usr/local/apr-util/bin/apu-1-config --version

-------------------------
1.6.1
```

### pcre 설치(컴파일/빌드)

{{< alert info no-icon >}}
PCRE(Pecl Compatible Regular Expressions) 는 펄 호환 정규 표현식으로서, 정규식 패턴 일치를 구현하는 함수의 집합입니다.
{{< /alert >}}

> 설치

```
cd /usr/local/src

wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz

tar xvfz pcre-8.44.tar.gz

cd pcre-8.44

./configure --enable-utf8 \
--prefix=/usr/local/pcre

make & make install
```

> 설치 확인

```
/usr/local/pcre/bin/pcre-config --version

---------------------------
8.44
```


### HTTP Server 다운로드

> Apache 다운로드 폴더 생성 및 이동

```shell
mkdir -p /usr/local/src/httpd && cd /usr/local/src/httpd
```

> Apache 다운로드

다운로드 주소: http://httpd.apache.org/download.cgi

```shell
wget https://downloads.apache.org//httpd/httpd-2.4.46.tar.gz
```

![다운로드 경로](/img/web/apache/http_server_download.png)

> 압축풀기

```shell
tar xvfz httpd-2.4.46.tar.gz
```





### HTTP Server 설치

/usr/local/src/httpd/httpd-2.4.46 폴더로 이동한 다음 아래 스크립트 실행

> 설치

```shell
# 이동
cd /usr/local/src/httpd/httpd-2.4.46

# 스크립트 수행
./configure \
--prefix=/data/apache2 \
--with-apr=/usr/local/apr \
--with-apr-util=/usr/local/apr-util \
--with-pcre=/usr/local/pcre \
--with-ssl=/usr/local/ssl \
--enable-all \
--enable-so \
--with-mpm=prefork

make && make install
```

> 기본 주속 수정

```
vi /data/apache2/conf/httpd.conf

--------------------------------

ServerName localhost:80

LoadModule ssl_module modules/mod_ssl.so
LoadModule rewrite_module modules/mod_rewrite.so

Include conf/extra/httpd-vhosts.conf
```

## Apache 시작

```
cd /data/apache2/bin

# 시작
./apachectl start

# 중지
./apachectl stop


# 포트 확인
netstat -lntp | grep 80
```

## Apache 웹서버 Service 등록

> vi /etc/systemd/system/httpd.service

```
[Unit]
Description=The Apache HTTP Server
After=syslog.target
After=network.target


[Service]
# 자식 프로세스 생성이 완료되는 단계까지를 systemd가 시작이 완료되었다고 판단하게 된다.
Type=forking
PIDFile=/data/apache2/logs/httpd.pid
#시작 명령을 정의
ExecStart=/data/apache2/bin/apachectl start
#reload 명령 정의
ExecReload=/data/apache2/bin/apachectl graceful
#종료 명령 정의
ExecStop=/data/apacher2/bin/apachectl graceful-stop
PrivateTmp=true
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
```

> 서비스 내용 추가 후 daemon-reload

```
systemctl daemon-reload
```

> 서비스 시작 및 재부팅시 실행 활성화

```
systemctl enable httpd

systemctl start httpd
```

> 서비스로그 확인

```
journalctl -u <서비스이름>

# -f: follow,
journalctl -f -u httpd
```