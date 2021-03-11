---
title: "Vagrant, Ansible를 활용한 WEB/WAS 구성 자동화"
date: 2021-03-11T16:32:08+09:00
draft: true
categories:
- infra
tags:
- Apache HTTP Server
- Tomcat
- mod_jk
#thumbnailImage: //example.com/image.jpg
author: "hyunwoo"
---
Code로 Server 생성하고, WEB/WAS 구성 자동화를 할 수 있을까?
<!--more-->


Getting Started
----------------------------------
이 데모는 최소한의 노력으로 WEB/WAS 환경을 구축하는 방법에 대한 안내를 제공 합니다.

다음 사항에 대하여 다룹니다.
 - Vargant 및 HyperV로 Server Provisioning
 - Ansible playbook 및 Host inventory 파일로 Application Provisioning
 - Apache HTTP Server와 Tomcat 연결
 - Tomcat Clustering 설정
 - Apache HTTP Server에 SSL 환경 구성하기.

Prerequisites
---------------------------
따라 하려면 다음 프로그램을 미리 사용 할 수 있어야 합니다.
 - Vagrant
 - Window 10 Hyper-V


Vagarnt 및 Hyper-v로 Server Provisioning 하기
-----------------------------------------------

![그림1](/img/infra/demo01/vagrant_server_provisioning.JPG)
**그림 1: Server Provisioning with Vagrant and HyperV.**