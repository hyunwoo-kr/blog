---
title: "File Server Provisioning 1차 - 설계"
date: 2021-04-16T19:22:44+09:00
categories:
- infra
tags:
- file-server
- rsync
- lsyncd
- File Server 실시간 백업
keywords:
- tech
thumbnailImage: /img/infra/file_server/thumbnail.JPG
author: "hyunwoo"

---
File Server 에 대해 실시간 백업을 하고, keepalived + NFS 도입하여, HA 구성까지 진행 해 보자.
<!--more-->

간략 시스템 구성도
---------------------------------

![그림1](/img/infra/file_server/file_server_system.JPG)

왼 쪽 그림은 평상시 <파일서버> 를 이용하는 구성이고, 오른쪽 그림은 <장애발생>시 <파일서버>를 이용하는 구성 입니다.

<장애발생>시 설명
-------------------------------

> ① 장애발생

File Server Master가 알 수 없는 이유로 shutdown 되었다고 가정

> ②, ③, ④ 장애감지 후 작업 - File Server Backup

File Server Backup에 설치된 keepavlied가 File Server Master의 장애를 감지하면,
② 동기화 중지하고,  ③ VIP 설정 하고, ④ NFS Server 기동 함

> ⑤ NFS Client

(추측) NFS Client들은 별도 작업 없이 [File Server]의 NFS 폴더에 접근 됨

핵심기능
-----------------------------------------

> File Server 동기화

    rsync와 lsyncd를 사용하여, 1초 간격으로 실시간 백업을 수행

> File Server HA 구성

    File Server는 keepalived를 사용하여 VIP(Virtual IP)를 만들고,
         Master 상태인 경우에만 NFS Server가 기동 되게 함

    NFS Client들은 File Server의 VIP 를 통해 NFS 구성을 함

그 다음 진행내용
-----------------------------

다음 글에서는 설계된 내용을 Vargrant로 Server를 Provisoning하고,
Ansible을 사용하여, 하나하나 구성하기
