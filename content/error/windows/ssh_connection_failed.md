---
title: "[VMware Connection failed] VMware SSH 연결 오류 - ip 구성이 올바르지 않습니다"
date: 2020-09-01
draft: false
category: [error]
subcategories: [windows]
tags: [Error, Windows, Xshell, VMware]
---

또 Vmware의 가상머신에 SSH로 연결하려 했더니 연결 오류가 났다.
지난번에는 오류 문구라도 있었는데, 이번에는 그냥 `Connection failed`다.  

<!--more-->

오류 상황은 아래와 같다.  

```sh
Connecting to 192.168.171.134:22...
Could not connect to '192.168.171.134' (port 22): Connection failed.

Type `help' to learn how to use Xshell prompt.
```

`ping` 명령어를 이용 해 봐도 제대로 응답이 오지 않길래, 내 Windows에서 `ipconfig`를 통해 네트워크를 확인 해 보았다.  

![](/images/error/ssh_connection_failed/01.png)  

확인 해 보니, 내 NAT로 구성되어 있는 내 가상머신들의 IP는 `192.168.171.0` 대역인데, VMware Network Adapter VMnet8은 `169.254.251.75`이다.
찾아보니 `169.254.x.x` 대역은 IP 주소가 없는 것과 동일하다고 볼 수 있다고 한다.
즉, 네트워크 연결이 안되는 것이다.
내 경우에는 어떤 이유인지는 모르겠지만 `VMNet8`의 대역을 못찾아 `169.254.251.75`로 나타난 것 같다.

이 경우 IP 주소 대역을 찾아 넣어주면 해결할 수 있다. 
`VMWare`에서 `Edit > Virtual Network Editor`를 선택 해 확인 해 보면 내 가상머신의 서브넷 주소를 알 수 있다.  

![](/images/error/ssh_connection_failed/02.png)  

이를 맞추기 위해서는 `Virtual Network Editor`를 변경해도 되는데, 나는 그냥 아래 명령어를 통해서 Windows에 설정을 변경했다.  

```sh
> ipconfig/renew
```

![](/images/error/ssh_connection_failed/03.png)  

그 결과 `VMware NEtwork Adapter VMnet8`의 IP 대역을 새로 설정하며 제대로 찾아 온 것을 알 수 있다.
다시 연결을 시도 해 보면 정상적으로 연결할 수 있다.  

참고로 내 경우, 이 이후로도 계속해서 IP를 못받아와 재부팅, ipconfig/renew 등을 했으나 같은 현상이 재현됬다.
그래서 결국 포맷하니 그 이후로는 현상이 재현되지 않았는데, 아무래도 네트워크 설정할 때 뭔가 꼬인 부분이 있었나보다.
~~PC가 너무 노후해서...?~~