---
title: "[SSH SOCKET ERROR] SSH Socket error Event: 32 Error: 10053"
date: 2020-08-10
draft: false
category: [error]
subcategories: [windows]
tags: [Error, Windows, Xshell, VMware]
---

Vmware에 새로운 가상머신 설치후 Xshell로 접속하려고 하니 `socket error`가 발생했다.
이 전에도 이런 일이 있었어서 해결 방법을 찾아 정리 해 둔다.

<!--more-->

오류 상황은 아래와 같다.  

```sh
Connecting to 192.168.171.142:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

Socket error Event: 32 Error: 10053.
Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(192.168.171.142:22) at 21:02:11.
```

그래서 이유를 확인하기 위해 `service ssh status`를 입력 했더니 아래와 같은 오류 메시지들을 확인할 수 있었다.  

![](/images/error/ssh_socket/01.png)  

ssh 연결에 사용되는 key들을 load할 수 없다는 오류였다.
권한 문제인가 해서 `/etc/sshd` 디렉터리로 들어가 key들을 확인 해 보았더니 아래와 같이 `size`가 `0`인 것을 확인할 수 있었다.  

![](/images/error/ssh_socket/02.png)  

이 경우 key들을 다시 재 생성 해 주면 된다.
나는 `ssh_host_dsa_key`, `ssh_host_ecdsa_key`, `ssh_host_ed25519_key`, `ssh_host_rsa_key`가 제대로 생성되지 않았으므로 아래의 명령어들을 통해 각각을 생성 해 주었다.  

```sh
$ ssh-keygen -f /etc/ssh/ssh_host_dsa_key -t dsa -N ""
$ ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -t ecdsa -N ""
$ ssh-keygen -f /etc/ssh/ssh_host_ed25519_key -t ed25519 -N ""
$ ssh-keygen -f /etc/ssh/ssh_host_rsa_key -t rsa -N ""
```

그럼 아래와 같이 출력되며 key 생성이 완료된다.  

![](/images/error/ssh_socket/03.png)  

다시 `/etc/sshd` 디렉터리를 확인해 보면 `size`가 `0`이 아닌 것을 확인할 수 있다.  

![](/images/error/ssh_socket/04.png)  

key를 다시 생성한 후 다시 SSH로 재접속을 해 보니 정상적으로 접속할 수 있었다.  