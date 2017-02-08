---
layout: post
title: 'Ubuntu VPN server'
tags: [ubuntu]
description: >
  Ubuntu VPN server setting
---
VPN(Virtual Private Network)은 IP를 우회할 수 있는 방법중에 하나이며 다양한 방법으로 응용가능하다. 예를들면 학교의 인터넷 망에서 자유롭게 논문을 볼 수 있지만(학교에서는 IEEE등에 논문을 구독할 수 있도록 지불을 했기 때문에) 외부 망에서는 동일하게 논문검색을 할 수 없다(물론 VPN을 이용하지 않고 논문을 검색하는 방법이 있긴 하다). 이럴 때 학교내 학교망에 연결되어 있는 컴퓨터에 VPN 설정을 하고 외부에서 이 컴퓨터 VPN server로 연결하여 외부로 접속을 한면 외부에서는 학교망에서 접속한 것으로 인식하기 때문에 학교내에서 접속한 것과 같은 효과를 볼 수 있다. Ubuntu 14.04에서 VPN server를 설정하는 방법은 다음과 같다.

## 1. pptpd 설치

```
sudo apt-get install pptpd
```

## 2. /etc/pptpd.conf 파일 수정

`/etc/pptpd.conf` 파일을 다음과 같이 설정한다. 이 아이피는 본인의 IP와는 상관없이 설정한다. 

```
localiip 10.0.01
remoteip 10.0.0.100-200
```

## 3. DNS 서버 추가
`/etc/ppp/pptpd-options` 에 다음을 추가한다. 

```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

## 4. IP forwarding을 활성화

`sudo vim /etc/sysctl.conf` 로 config 파일을 열고 다음을 활성화(uncomment) 시킨다(주석을 제거 하라는 의미).

```
net.ipv4.ip_forward = 1
```

## 5. 변경 저장


```
sudo sysctl -p /etc/sysctl.config
```

## 6. VPN user 등록

`/etc/ppp/chap-secrets`을 열어 다음과 같이 user를 등록해 준다.

```
# Secrets for authentication using CHAP
# client    server  secret          IP addresses
username    pptpd    password              *
```

IP addresses에는 그냥 *를 입력한다.

## 7. 재 시작

`service pptpd restart`를 입력하여 재 시작한다. 

설정 상태는 `ifconfig`를 입력하여 확인 할 수 있다.

## 8. iptable 변경

`iptables-save > ~/iptables.save`를 입력하여 iptable을 백업한다.  그 다음으로 변경된 iptable을 기본 인터페이스로 사용하기 위해 다음을 입력한다(sudo로 입력해야 할 수도 있다.)

```
iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE
iptables --table nat --append POSTROUTING --out-interface ppp0 -j MASQUERADE
iptables -I INPUT -s 10.0.0.0/8 -i ppp0 -j ACCEPT
iptables --append FORWARD --in-interface br0 -j ACCEPT
```

이 설정을 reboot 후에도 유지하기 위해서 다음을 입력한다.

```
sudo apt-get install iptables-persistent
```

## 9. 옵션 설정

VPN 연결을 위해 Ubuntu 14.04를 사용하고 있다면, PPTP Advanced Option에서 (use Point to point encryption)을 클릭해야 한다. 
