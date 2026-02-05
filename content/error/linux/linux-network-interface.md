---
title: "[Linux 오류] Cannot find device ens33"
date: 2018-11-18
draft: false
category: [error]
subcategories: [linux]
tags: [Error, Linux]
---

고정 IP를 사용하던 Ubuntu 가상머신을 다시 동적 IP를 사용하도록 변경해야 하는 일이 있었다.  
그런데 아무리 설정을 변경해도 네트워크를 시작할 수 없었다. 이에 대한 해결 방법을 적어두려 한다.  

<!--more-->

고정 IP를 사용할 지, 동적 IP를 사용할 지 설정하기 위해서는 `/etc/network/interfaces` 파일을 수정하면 된다. 보통 동적 IP를 사용할 때는 아래와 같다.  

```sh
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

#source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto ens3
iface ens33 inet dhcp
```

정적 IP를 사용하고 싶다면 아래와 같이 설정 부분을 수정하면 된다.  

```sh
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

#source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto ens3
iface ens33 inet static
address YOUR_IP_ADDRESS
netmask TOUR_NETMASK
gateway YOUR_GATEWAY
dns-nameservers YOUR_DNS_NAMESERVER
```

이 후, `systemctl restart networking.service`를 하면 IP가 원하는데로 바뀌어 있는 것을 확인할 수 있다.
그런데, 나는 고정 IP를 사용하던 VM을 동적 IP를 사용하도록 변경해야 하는 상황이었다.

그래서 맨 위의 파일의 내용으로 `/etc/network/interfaces`를 수정한 후, `systemctl restart netwoking.service`를 했더니 계속해서 오류만 났다.
이 때, 해당 서비스를 재시작 할 수 없는 이유를 알기 위해서는 `systemctl status netwoking.service` 명령어를 사용하면 된다.
위의 명령어를 확인 해 보니 아래와 같은 로그들이 남아있었다.  

```sh
Unknown interface ens33

...

Cannot find device "ens33"
```

내가 /etc/network/interfaces/에 적어 둔 ens33이라는 인터페이스가 없다는 오류였다.
내 서버의 interface를 확인하기 위해서는 아래와 같은 명령어를 사용하면 된다.  

```sh
$ lshw -C network
```

그럼 network와 관련 된 정보가 쭉 뜨는데, 이 중 `logical name`을 확인 해 보면 된다.
내 VM의 경우, ens33이 아니라 ens34를 사용하고 있었다.
이에, `/etc/network/interfaces`에서 ens33을 ens34로 변경 해 주고 다시 network를 재시작 했더니 문제를 해결할 수 있었다.  