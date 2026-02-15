---
title: "Windows10 Hyper-V Nat Network 구성하기"
date: 2021-03-04T16:42:05+09:00
categories:
- infra
tags:
- Windows 10
- hyper-v
- nat network
- PowerShell
keywords:
- hyper-v
- network set
---
Windows 10, Hyper-v 를 사용할 때 Network 구성을 정형화 할 수 있을까?
<!--more-->
[목표]
- Hyper-V 네트워크를
- 명령어 기반, Copy&Paste 로
- 인터넷이 되는 <DMZ망>과
- 인터넷이 안되는 <폐쇠망>을 구성

[기반 기술]
- PowerShell 명령어

---

## Hyper-V NAT 가상 네트워크 만들기

 출처: [setup-nat-network](https://docs.microsoft.com/ko-kr/virtualization/hyper-v-on-windows/user-guide/setup-nat-network)

 - 내부 스위치 생성
 - NAT 게이트웨이를 구성
 - vagarnt로 CentOS 7 Server를 생성하고 네트워크 연결하기

{{< alert danger >}}
관리자 권한으로 PowerShell 콘솔 오픈 후 아래 명령을 실행해야 함.
{{< /alert >}}

> 내부(Internal) 스위치 생성

[문법]
```
# PowerShell. 내부(Internal) 스위치 생성
New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
```

[예시]
```
New-VMSwitch -SwitchName "dmz" -SwitchType Internal
```

![실행결과](/img/hyper-v/result01.jpg)

> 방금 만든 가상 스위치의 인터페이스 색인 확인 (ifIndex 를 적어 두기)

```
C:\> Get-NetAdapter -Name "vEthernet (dmz)"
```
![실행결과](/img/hyper-v/result02.jpg)
위 그림에서 빨간색 네모난 부분

> New-NetIPAddress를 사용하여 NAT 게이트웨이를 구성합니다.


```
New-NetIPAddress -IPAddress 161.100.6.1 -PrefixLength 24 -InterfaceIndex 63
```


![실행결과](/img/hyper-v/result03.jpg)
 - IP는 161.100.6.1을 사용.
 - 위 그림에서 빨간색 네모의 값은 -InterfaceIndex 의 값으로 조회 했던 ifIndex 값 <63> 으로 입력 했음

> New-NetNat를 사용하여 NAT 네트워크를 구성합니다.

[문법]
    ```
    New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
    ```

[예시]
```
New-NetNat -Name dmz -InternalIPInterfaceAddressPrefix 161.100.6.0/24
```

게이트웨이를 구성하려면 <네트워크> 및 <NAT 게이트웨이>에 대한 정보를 제공해야 합니다.

- NATOutsideName은 `NAT 네트워크`의 `이름` 을 설명. 이 예제 에서는 `dmz` 를 사용함

- InternalIPInterfaceAddressPrefix 은 `NAT 서브넷 접두사`로  `NAT 게이트웨이 IP 접두사`와 `NAT 서브넷 접두사 길이`를 모두 설명
- 일반적인 형식은 a.b.c.0/NAT 서브넷 접두사 길이입니다. 이 예제에서는 위의 `161.100.6.0/24`  사용