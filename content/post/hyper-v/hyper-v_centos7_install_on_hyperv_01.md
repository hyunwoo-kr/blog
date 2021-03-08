---
title: "Apache HTTPD 테스트용 Centos7 구성 - 1차"
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
[목표]
- Hyper-V Nat 네트워크 구성을 활용하여
- CentOS7을 Hyper-V에 설치 해 보자

[사전준비]
 - [Windows10 Hyper-V Nat Network 구성하기](/2021/03/windows10-hyper-v-nat-network-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/)
 - Vagrant 설치 및 사용법 숙지

## Vagrant 로 Hyper-v에 Centos7 프로비저닝하기

{{< alert danger no-icon >}}
반드시, Windows PowerShell의 관리자 권한으로 아래 명령을 수행 해야 함
{{< /alert >}}

### Step1 Vagrant 설치

{{< alert success no-icon >}}
[Download Vagrant](https://www.vagrantup.com/downloads)
{{< /alert >}}

![download_vagrant](/img/hyper-v/vagrant01/download_vagrant.JPG)

> 설치 확인

```
vagrant -version
```

![vagrant_check](/img/hyper-v/vagrant01/vagrant_check.JPG)

### Step2 Hyper-V용 centos/7 box 다운로드 하기

{{< alert success no-icon >}}
다운로드 주소: https://app.vagrantup.com/centos/boxes/7
{{< /alert >}}

![다운로드](/img/hyper-v/vagrant01/vagrant_box_download.png)


### Step3 다운로드한 box add 하기

{{< alert success no-icon >}}
1) q 드라이브,
2) file:///Q:\vs_work\infra\vagrant_box\ 폴더에
3) CentOS-7-x86_64-Vagrant-2004_01.HyperV.box 파일을 받았다고 가정.
{{< /alert >}}

```
vagrant box add centos/7 file:///Q:\vs_work\infra\vagrant_box\CentOS-7-x86_64-Vagrant-2004_01.HyperV.box
```

### Step4 Vagrant 프로젝트 생성

![폴더구조](/img/hyper-v/vagrant01/project_dir.JPG)

 - Vagrantfile: 베이그런트 이미지 설정파일
 - configure-static-ip.sh: 사설 고정IP 할당 스크립트

### Step5 Vagrantfile 생성

> Copy&Paste를 진행하고, <비밀번호>만 사용하고 싶은 걸로 대체한다.
>
> <비밀번호>는 root 패스워드로 설정된다

```
Vagrant.configure("2") do |config|
    config.vm.box = "centos/7"
    config.vm.box_check_update = true
    config.vm.provider "hyperv"
    config.vm.network "public_network", auto_config: false,  bridge: "dmz"
    config.vm.hostname = "dzm-web1"

    config.vm.synced_folder ".", "/vagrant"

    config.vm.provider "hyperv" do |vb|
        vb.auto_start_action= "StartIfRunning"
        vb.auto_stop_action = "TurnOff"
        vb.cpus = 1
        vb.memory = 1024*2
        vb.vmname = "dzm-web1"
    end

    config.vm.provision "shell", inline: <<-SHELL
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config
        sudo su && echo "<비밀번호>" | passwd --stdin root
    SHELL

    config.vm.provision "shell", inline: <<-SHELL
        /vagrant/scripts/configure-static-ip.sh
        shutdown -h now
    SHELL

    config.trigger.after :up  do |trigger|
        trigger.info = "vm shutdown wating 10 seconds"
        trigger.run = {inline: "sleep 10"}
    end

    config.trigger.after :up  do |trigger|
        trigger.info = "START VM"
        trigger.run = {inline: "Start-VM -Name dzm-web1"}
    end
end
```


### Step6 configure-static-ip.sh

> 고정IP 161.100.6.10 으로 설정

```
#!/bin/sh

echo 'Setting static IP address for Hyper-V...'

cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
PREFIX=24
IPADDR=161.100.6.10
GATEWAY=161.100.6.1
DNS1=8.8.8.8
EOF
```

### Step7 hyper-v 로  CentOS/7 만들기

{{< alert danger no-icon>}}
1) 반드시, PowerShell 관리자 모드
2) /2_centos7_install/vagrant 폴더로 이동 후
3) 아래 명령어를 수행 해야 함
{{< /alert >}}

```shell
vagrant up
```
![result](/img/hyper-v/vagrant01/final_result.png)

![result_hyperv](/img/hyper-v/vagrant01/result_hyperv.JPG)


### Step8 Console 접속

```
ssh root@161.100.6.10
```

## Step9 OS 운영체제 및 버전확인

```
grep . /etc/*-release
```
![os_version](/img/hyper-v/vagrant01/os_version.JPG)

다음에는 이 CentOS7을 Ansible 을 사용하여 Apache HTTPD 를 설치하는 방법을 정리해야지.